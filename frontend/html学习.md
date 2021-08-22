# html学习

## form表单

form有一个元素 -- action，action有三种模式分别是text、radio、submit

| 类型   | 描述                         |
| ------ | ---------------------------- |
| text   | 定义常规文本输入             |
| radio  | 定义单选按钮                 |
| submit | 定义提交按钮（提交表单信息） |

示例：

1. text：

```html
<form>
    First Name:<br>
    <input type="text" name="firstname">
    <br>
    Last Name:<br>
    <input type="text" name="lastname">
</form>
```

![](E:\workspace\document\frontend\resources\form\formText.png)

2. radio

```html
<form>
    <input type="radio" name="sex" value="male" checked>Male
    <br>
    <input type="radio" name="sex" value="female">Female
</form> 
```

![](E:\workspace\document\frontend\resources\form\formradio.png)

3. submit

```html
<form action="action_page.php">
    First name:<br>
    <input type="text" name="firstname" value="Mickey">
    <br>
    Last name:<br>
    <input type="text" name="lastname" value="Mouse">
    <br><br>
    <input type="submit" value="Submit">
</form> 
```

![](E:\workspace\document\frontend\resources\form\formsubmit.png)