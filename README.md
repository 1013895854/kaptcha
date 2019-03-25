# kaptcha - A kaptcha generation engine.

This repo is the copy of http://code.google.com/p/kaptcha/ and published to maven central
```
<dependency>
  <groupId>com.github.penggle</groupId>
  <artifactId>kaptcha</artifactId>
  <version>2.3.2</version>
</dependency>
```
Please see the website for more information about this project.

http://code.google.com/p/kaptcha/

thanks!

springboot 集成 Kaptcha
首先导入jar包
1.配置类
@Configuration
public class KaptchaConfig {
	@Bean
	public DefaultKaptcha getDefaultKaptche() {
		 DefaultKaptcha captchaProducer = new DefaultKaptcha();
	        Properties properties = new Properties();
           // 是否有边框  默认为true  我们可以自己设置yes，no
	        properties.setProperty("kaptcha.border", "yes");
          // 边框颜色   默认为Color.BLACK  
	        properties.setProperty("kaptcha.border.color", "105,179,90");
           // 验证码文本字符颜色  默认为Color.BLACK 
	        properties.setProperty("kaptcha.textproducer.font.color", "blue");
           // 验证码图片宽度  默认为200
	        properties.setProperty("kaptcha.image.width", "110");
           // 验证码图片高度  默认为200
	        properties.setProperty("kaptcha.image.height", "40");
           // 验证码文本字符大小  默认为40
	        properties.setProperty("kaptcha.textproducer.font.size", "30");
          // KAPTCHA_SESSION_KEY
	        properties.setProperty("kaptcha.session.key", "code");
          // 验证码文本字符间距  默认为2
          properties.setProperty("kaptcha.textproducer.char.space", "3");
           // 验证码文本字符长度  默认为5
	        properties.setProperty("kaptcha.textproducer.char.length", "4");
          // 验证码文本字体样式 
	        properties.setProperty("kaptcha.textproducer.font.names", "宋体,楷体,微软雅黑");
	        Config config = new Config(properties);
	        captchaProducer.setConfig(config);
	        return captchaProducer;
		
	}
}

2.验证验证码是否正确的工具类
public class CodeUtil {
    /**
     * 将获取到的前端参数转为string类型
     * @param request
     * @param key
     * @return
     */
    public static String getString(HttpServletRequest request, String key) {
        try {
            String result = request.getParameter(key);
            if(result != null) {
                result = result.trim();
            }
            if("".equals(result)) {
                result = null;
            }
            return result;
        }catch(Exception e) {
            return null;
        }
    }
    /**
     * 验证码校验
     * @param request
     * @return
     */
    public static boolean checkVerifyCode(HttpServletRequest request) {
        //获取生成的验证码
        String verifyCodeExpected = (String) request.getSession().getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);
        //获取用户输入的验证码
        String verifyCodeActual = CodeUtil.getString(request, "verifyCodeActual");
        if(verifyCodeActual == null ||!verifyCodeActual.equals(verifyCodeExpected)) {
            return false;
        }
        return true;
    }
}

3.Controller 的样式
@Controller
public class KaptchaController {
	 @Autowired
	    private Producer captchaProducer = null;
	    @RequestMapping("/kaptcha")
	    public void getKaptchaImage(HttpServletRequest request, HttpServletResponse response) throws Exception {
	        HttpSession session = request.getSession();
	        response.setDateHeader("Expires", 0);
	        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");
	        response.addHeader("Cache-Control", "post-check=0, pre-check=0");
	        response.setHeader("Pragma", "no-cache");
	        response.setContentType("image/jpeg");
	        //生成验证码
	        String capText = captchaProducer.createText();
	        session.setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);
	        //向客户端写出
	        BufferedImage bi = captchaProducer.createImage(capText);
	        ServletOutputStream out = response.getOutputStream();
	        ImageIO.write(bi, "jpg", out);
	        try {
	            out.flush();
	        } finally {
	            out.close();
	        }
	    }
	    
	    @RequestMapping("/hello")
	    @ResponseBody
	    public String hello(HttpServletRequest request) {
	        if (!CodeUtil.checkVerifyCode(request)) {
	            return "验证码有误！";
	        } else {
	            return "hello,world";
	        }
	    }
}

5.前台书写样式
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript">
        function refresh() {
            document.getElementById('captcha_img').src="http://localhost:8080/kaptcha?"+Math.random();
        }
    </script>
</head>
<body>
<form action="http://localhost:8080/hello" method="post">
    验证码:  <input type="text" placeholder="请输入验证码" name="verifyCodeActual">
    <div class="item-input">
        <img id="captcha_img" alt="点击更换" title="点击更换"
             onclick="refresh()" src="http://localhost:8080/kaptcha" />
    </div>
    <input type="submit" value="提交" />
</form>

</body>
</html>
