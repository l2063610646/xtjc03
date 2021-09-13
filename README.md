# xtjc03## java通过串口控制蜂鸣器
******

### 1、定义实体类Beep
```java
public class Beep {
    
    //单片机响应时间
    private String currentDate;
    
    //命令状态
    private String cmdStatus;
    
    //设备状态
    private String deviceStatus;

    public String getCurrentDate() {
        return currentDate;
    }

    public void setCurrentDate(String currentDate) {
        this.currentDate = currentDate;
    }

    public String getCmdStatus() {
        return cmdStatus;
    }

    public void setCmdStatus(String cmdStatus) {
        this.cmdStatus = cmdStatus;
    }

    public String getDeviceStatus() {
        return deviceStatus;
    }

    public void setDeviceStatus(String deviceStatus) {
        this.deviceStatus = deviceStatus;
    }

    @Override
    public String toString() {
        return "蜂鸣器{" +
                "响应时间=" + currentDate +
                ", 命令状态=" + cmdStatus +
                ", 设备状态=" + deviceStatus +
                '}';
    }
}
```

### 2、定义Service接口
```java
public interface BeepService {

    Beep start();

    Beep stop();

    Beep read();

}

```
### 3、修改CommUtil类
![img.png](img.png)

#### 这里定义了两个静态属性,一个是result一个是flag

- flag：串口返回消息的标志位
- result：串口返回的消息
```java
public class Result{

    private String data;

    private String time;

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }

    public String getTime() {
        return time;
    }

    public void setTime(String time) {
        this.time = time;
    }
}
```

![img_1.png](img_1.png)
```java
修改串口数据处理的方法，将接收到的data存到result里
```

### 4、实现Service接口
```java
@Service
public class BeepServiceImpl implements BeepService{
    
    //向单片机发送指令时，需要在前面携带xtjc03，否则单片机不做处理
    private static final String COMPANY = "xtjc03";

    @Autowired  //自动装配
    CommUtil commUtil;

    @Async  //异步任务
    @Override
    public Beep start() {
    	CommUtil.flag = false;
        commUtil.send(COMPANY + "01\r\n");  //记得换行！！
        Beep comMsg = getComMsg("打开蜂鸣器", 3000);
        CommUtil.flag = false;
        return comMsg;
    }

    @Async  //异步任务
    @Override
    public Beep stop() {
    	CommUtil.flag = false;
        commUtil.send(COMPANY + "00\r\n");  //记得换行！！
        Beep comMsg = getComMsg("关闭蜂鸣器", 3000);
        CommUtil.flag = false;
        return comMsg;
    }

    @Async  //异步任务
    @Override
    public Beep read() {
        CommUtil.flag = false;
        commUtil.send(COMPANY + "02\r\n");  //记得换行！！
        Beep comMsg = getComMsg("读取状态", 3000);
        CommUtil.flag = false;
        return comMsg;
    }

    /**
     * 获取串口返回的数据
     * @param msg
     * @param delay
     * @return
     */
    public Beep getComMsg(String msg, long delay) {
    	Beep beep = new Beep();
    	beep.setCmdStatus(msg);
    	long beginTime = System.currentTimeMillis();
    	while (System.currentTimeMillis() - beginTime <= delay){
            if (CommUtil.flag){
                Result result = CommUtil.result;
                String data = result.getData();
                String time = result.getTime();
                beep.setCurrentDate(time);
                beep.setDeviceStatus(data.equals(COMPANY + ":1") ? "开" : "关");
                break;
            }
            try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
        }
    	return beep;
    }
}

```

### 5、编写控制器
```java
@RestController
@RequestMapping(value="/api/beep")
public class BeepController {

    @Autowired
    BeepServiceImpl beepService;

    @Value("${server.port}")
    String serverPort;

    @GetMapping("/start")
    public Beep start(){
        return beepService.start();
    }

    @GetMapping("/stop")
    public Beep stop(){
        return beepService.stop();
    }

    @GetMapping("/read")
    public Beep read(){
        return beepService.read();
    }

    @GetMapping("/")
    public Map<String, String> route(){
        Map<String, String> map = new HashMap<>();
        map.put("打开蜂鸣器", "http://127.0.0.1:" + serverPort + "/api/beep/start");
        map.put("关闭蜂鸣器", "http://127.0.0.1:" + serverPort + "/api/beep/stop");
        map.put("读取蜂鸣器状态", "http://127.0.0.1:" + serverPort + "/api/beep/read");
        return map;
    }
}
```
