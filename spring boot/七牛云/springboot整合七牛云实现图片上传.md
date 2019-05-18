之前有在springboot上用过七牛云实现图片上传，今天因为某些原因又重新使用了下七牛云因此想总结下七牛云

# 详细步骤
## 1.申请七牛云账号并实名认证
## 2.申请存储空间

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518151032106.png)

完善存储空间名并选择地区

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518151104691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

## 3.springboot整合七牛云实现图片上传

1.导入maven依赖
```xml
        <!-- 七牛云 -->
        <dependency>
            <groupId>com.qiniu</groupId>
            <artifactId>qiniu-java-sdk</artifactId>
            <version>[7.2.0, 7.2.99]</version>
        </dependency>
```

2.获取Access_Key,和SECRET_KEY

可以到七牛云控制台个人中心密钥管理获取

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518151728468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

3.编写七牛云工具类实现图片上传

```java
@Slf4j
public class QiniuUtils {

    // 设置需要操作的账号的AK和SK
    private static final String ACCESS_KEY = TestQiniuConfig.AK;
    private static final String SECRET_KEY = TestQiniuConfig.SK;

    // 要上传的空间名称
    private static final String BUCKETNAME = TestQiniuConfig.BUCKET;

    // 密钥
    private static final Auth auth = Auth.create(ACCESS_KEY, SECRET_KEY);

    // 外链默认域名
    private static final String DOMAIN = TestQiniuConfig.DOMAIN;

    /**
     * 将图片上传到七牛云
     */
    public static String uploadQNImg(FileInputStream file, String key) {
        // 构造一个带指定Zone对象的配置类, 注意这里的Zone.zone0需要根据主机选择
        Configuration cfg = new Configuration(Zone.zone2());
        // 其他参数参考类注释
        UploadManager uploadManager = new UploadManager(cfg);
        // 生成上传凭证，然后准备上传

        try {
         //    Auth auth = Auth.create(ACCESS_KEY, SECRET_KEY);
            String upToken = auth.uploadToken(BUCKETNAME);
            try {
                Response response = uploadManager.put(file, key, upToken, null, null);
                // 解析上传成功的结果
                DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);

                String returnPath = DOMAIN + "/" + putRet.key;
                // 这个returnPath是获得到的外链地址,通过这个地址可以直接打开图片
                return returnPath;
            } catch (QiniuException ex) {
                Response r = ex.response;
                System.err.println(r.toString());
                try {
                    System.err.println(r.bodyString());
                } catch (QiniuException ex2) {
                    //ignore
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }
}
```

关于地区的编码 可查看官方文档[七牛云](https://developer.qiniu.com/kodo/manual/1671/region-endpoint)

```java
        // 构造一个带指定Zone对象的配置类, 注意这里的Zone.zone0需要根据主机选择
        Configuration cfg = new Configuration(Zone.zone2());
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518153236593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


4.controller层编写
```java
@Slf4j
@Controller
@RequestMapping("/qiniu")
public class TestQiNiuController {

    @RequestMapping("/toIndex")
    public String index(){
        return "img";
    }

    @ResponseBody
    @RequestMapping(value = "/image", method = RequestMethod.POST)
    private String postUserInforUpDate(HttpServletRequest request, @RequestParam("file") MultipartFile multipartFile) throws IOException {

        // 用来获取其他参数
        MultipartHttpServletRequest params = ((MultipartHttpServletRequest) request);
        String name = params.getParameter("username");
        if (!multipartFile.isEmpty()) {
            FileInputStream inputStream = (FileInputStream) multipartFile.getInputStream();
            String path = QiniuUtils.uploadQNImg(inputStream, PictureUtil.getRandomFileName()); // KeyUtil.genUniqueKey()生成图片的随机名
            System.out.print("七牛云返回的图片链接:" + path);
            return path;
        }
        return "上传失败";
    }
}
```

图片上传成功后，idea控制台会返回一个图片的外网链接，前台可以通过这个外网链接获取到图片显示到前台

关于图片访问的外网链接是拼接出来的,通过解析上传成功的结果和七牛云配置的外网链接（默认是http://prom5oysi.bkt.clouddn.com）
```java
                Response response = uploadManager.put(file, key, upToken, null, null);
                // 解析上传成功的结果
                DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
                String returnPath = DOMAIN + "/" + putRet.key;
                // 这个returnPath是获得到的外链地址,通过这个地址可以直接打开图片
                return returnPath;

```

5.前台html和js
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>demo</title>
    <link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">
    <script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
    <script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<script>
    /**
     * 上传图片
     */
    function uploadImg(){

        /**
         * formData在jquey中使用需要设置：
         * processData: false, // 告诉jQuery不要去处理发送的数据
         * contentType: false // 告诉jQuery不要去设置Content-Type请求头
         * @type {null}
         */
        var fd = new FormData();

        // 第一个参数为controller 接收的参数名称 , input的id
        fd.append("file", document.getElementById("inputId").files[0]);
        $.ajax({
            url:"http://localhost:8082/all/qiniu/image",
            type:"post",
            data:fd,
            processData:false,
            contentType:false,
            success:function(res){
                console.log(res);

                if (res.status.code == 0) {
                    if (!$('#img').empty()) {
                        $('#img').empty();
                    }
                    // 这一串代码复制不上来 ,截图在下面
                    $('#img').append(" ![](+res.result[0]+)");
                } else {
                    alert("图片上传失败");
                }
            },
            dataType:"json"
        })
    }
</script>

<div id="img">
    <input type="file" name="text" id="inputId">
    <input type="submit" onclick="uploadImg()">
    <!-- 图片链接 -->
    <img src="http://prom5oysi.bkt.clouddn.com/pciture9366020190518" width="1000px">
</div>

</body>
</html>
```

6.控制台查看上传结果

上传完成后可以到七牛云控制台查看上传结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518152733215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)