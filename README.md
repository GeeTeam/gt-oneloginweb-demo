**概述与资源**

以下是OneLogin API H5的部署文档，用于指导 OneLogin API H5的集成。

**部署**

# H5一键登录 (OneLogin API)

### OneLogin API 推荐逻辑
1 页面引用 oneloginh5.js 

2 请在head中添加代码（生产上有https访问的，会导致上报的referer为空，移动运营商会去校验请求referer是否进行备案）
```html
  <meta content="always" name="referrer">
```
3 初始化OneLogin API对象
<span style="color:red">
为了避免影响认证的响应时间，可以先判断当前网络环境（可参考H5onelogindemo中的方法）再进行相对初始化；建议开发者在页面加载的时候判断网络环境并初始化。
</span>
字段说明：

| 字段         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| app_id       | 客户在客户后台申请的。审核成功后生效                         |
| timeout      | 超时时长                                                     |
| logo         | 授权页面logo展示（绝对路径地址、尺寸80*80）                                                   |

```javascript
 var olInstance = new GOL({            
    app_id: '您的app_id', // 在后台获取，生成app_id后等待审核成功后生效
    timeout: 10000, // 超时设置 默认30000
  });
```

4 onTokenSuccess、onTokenFail 获取token参数后去校验是否为本机

```javascript
 olInstance.onTokenSuccess(function(data){
    // data 参数包含 token、phone、process_id
    // 调用服务端校验接口， 以下是伪代码示例
    axios({
      method: 'POST',
      url: '您API地址',// 您的服务端校验接口
      data: `校验参数` //        
    }).then(res=>{
      if(res.data && res.data.status === 200){
        // success 获取返回结果
      } else {
        // fail， 可以调用短信等其他验证形式
      }
    });

  }).onTokenFail(function(){      
    //  网关失败，可以调用短信等其他验证形式
  })
```

5 在需要使用H5一键登录时调用gateway接口，去运营商请求获取token, 成功/失败将在onTokenSuccess、onTokenFail中返回

```javascript
 olInstance.gateway() 
```

 <span style="color:red">
 H5onelogindemo.html中是一个示例代码演示，详细演示了各个部署步骤和失败后降级处理方法。
 您可以将它导入到您的工程，按照demo中的步骤进行把参数填入后即可运行起来。</span>

### 方法说明

**gateway(options)** 调用网关接口，去运营商获取token
>fn 调用网关接口 去运营商取号

**onTokenSuccess(fn)**运营商取号token成功返回

>fn 成功返回函数，返回函数参数是Object, 结构：{ process_id: 'xxxx', phone:'', token: 'abc' }。

**onTokenFail (fn)** 运营商取号token失败返回

>fn 失败返回函数,返回函数参数是Object, 结构：{ code: 10010 }



