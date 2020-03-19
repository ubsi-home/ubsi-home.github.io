# rewin.ubsi.consumer.Logger 日志

---



```
    /** 输出DEBUG日志 */
    public void debug(String tips, Object data);
    
    /** 输出INFO日志 */
    public void info(String tips, Object data);
    
    /** 输出WARN日志 */
    public void warn(String tips, Object data);
    
    /** 输出ERROR日志 */
    public void error(String tips, Object data);
    
    /** 输出ACTION日志 */
    public void action(String tips, Object data);
    
    /** 输出ACCESS日志 */
    public void access(String tips, Object data);
    
    /** 输出APP自定义级别日志 */
    public void log(int type, String tips, Object data);
```

