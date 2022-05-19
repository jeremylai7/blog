# SpringBoot统一异常处理

```
@ControllerAdvice
public class ExceptionHandle {

     private final static Logger logger = LoggerFactory.getLogger(ExceptionHandle .class);
   
     @ExceptionHandler(value = Exception.class)
     @ResponseBody
     public Result handle(Exception e){
           Result result = OutUtil.error(e.getCode(), e.getMessage());
           logger.error("business exception:" + result.getMessage() + "");
           return result; 
     }      
    
}
```
