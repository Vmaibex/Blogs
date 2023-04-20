## 熟悉前端项目的个人记录
项目是一个简单的个人博客系统，包括管理端和访客端。

### 公共部分

这又分主要是介绍封装 axios，用于发起请求的 Request 脚本，更有助于理解在前端项目中如何对后端发起请求。



### 管理端

管理端的主要功能有：用户登录，新增博客，新增分类，新增专题，个人信息设置，用户管理，系统设置和回收站。接下来的内容是记录每一个功能的实现流程，供个人面试时使用。

#### 用户登录

关于登录界面，将从界面布局和登录功能两方面记录。登录界面的设计比较简单，外观如下：

![image-20230420131358381](.\assets\image-20230420131358381.png)

##### 界面布局

在界面组件的选择上，我们选择了 element-plus 组件库来提升整体的美观程度。我们的登录界面包含一张大的背景图片和一个用户登录 panel，主要布局都在登录 panel这边。首先记录比较简单的大图背景，这部分的 css用到了如下代码：

```css
width: 100%;		// % 用来定义基于包含块（父元素）宽度的百分比
height: calc(100vh);	// calc函数用于动态计算长度值，vh的意思是 viewpoint height，视窗高度，1vh=视窗高度的1% 
background-size: cover;		// 在background-size中 cover为保持图像的纵横比并将图像缩放成将完全覆盖背景定位区域的最小大小。
background-position: center;	// 设置背景图像的起始位置。
background-image: url('文件路径')；
```

我们在代码中的每一行都进行了注释，来解释这一行样式的作用。

之后是用户登录 panel，这一部分包含用户登录和一个el-form表单块，每个el-form中包含几个el-form-item组件，给出这一部分的 css 代码：

```css
.login-panel{
    float: right;	// 用户登录 panel 浮于整个界面右侧
    width: 350px;	// 设置 panel 的宽度为 350 像素，固定值
    margin-right: 100px;	// 为了与界面右侧保持一定的宽度
    padding: 20px;
    margin-top: 150px;
    border-radius: 5px;		// 设置圆角
    background: rgba($color: #fff, $alpha: 0.8);	// 设置背景透明
    box-shadow: 2px 2px 10px #ddd;		// 设置盒子阴影

    .login-title{
        font-size: 20px;
        text-align: center;
        margin-bottom: 10px;
    }

    .check-code-panel{
        width: 100%;
        display: flex;
        align-items: center;
        .input-panel{
            flex: 1;
            margin-right: 10px;

        }
        .check-code{
            // height: 30px;
            // margin-right: 2px;
            cursor: pointer;		// 定义了鼠标指针放在一个元素边界范围内时所用的光标形状， pointer为手型
        }
    }
}
```

接下来介绍逻辑功能，可以参考 element-plus 官网的 form 表单校验部分，这里只详细记录界面用户登录、记住密码等的逻辑流程，这里用到的第三方库是vue-cookie，用到的密码加密工具是 md5。主要逻辑如下：

界面初始化的时候，会使用vue cookie 检查有没有先前的没过期的登录信息 logInfo，如果有，我们则把表单中的用户名、密码、以及是否记住密码信息赋值，如果没有则跳过，正常初始化。然后是点击登录按钮时触发的逻辑，首先是表单的校验，如果当前表单合法，则进行后续操作，如果不合法，则会提示用户表单不合法；当表单合法时，首先会从 logInfo 中获取信息，并提取 logInfo 中的密码，如果 logInfo 不存在，则密码为空，这时检查 表单中的密码和 logInfo 中的密码是否一致，如果不一致，则证明密码是最新输入的， 需要对密码进行 md5 的加密。然后将登录参数设置完毕，包括用户名，密码和随机生成的验证码。如果都正确则证明登录成功，否则不成功，会提示用户。成功之后，会设置 1.5s 后进行页面跳转，然后，将登录接口的返回信息作为对象 存到 另外一个cookie中 uerinfo，表示当前会话的用户是哪一位，并且在这时判断是否记住密码，如果记住，则将当前用户的登录信息保存7天，不记住的话 loginfo 则没有信息。

在这里需要记住两个存到cookie中的信息，一个是 logInfo，用于记录用户的登录信息，是跟**是否记住密码**相关联的，记住密码意味着下次登录的时候就能获取先前的登录信息，就可以使用 logInfo，这个的存活时间是7天；另外一个是 userInfo，这个是用于记录当前会话的用户是谁，其存的信息是登录方法的返回值，用于后续判断博客是否归当前用户所有，以实现 **区别不同用户行为**的功能，这个的存活时间是当前浏览器会话结束。

