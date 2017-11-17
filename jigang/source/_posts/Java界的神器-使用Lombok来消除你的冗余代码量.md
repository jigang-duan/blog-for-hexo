---
title: Java界的神器-使用Lombok来消除你的冗余代码量
date: 2017-10-09 11:23:46
tags:
- Java
categories:
- Android优化

---


## 简介

他是一个通过注解方式来减少你的POJO类的getter和setter等方法的一个工具，我这里演示的在Android Studio中的使用方式，当然如果你使用的是idea那么这方法也通用，如果你用的是eclipse，那么[官网](https://projectlombok.org/)也有视频教程，我这里就不演示了

<!-- more -->

## 安装依赖

众所周知在在Android Studio中添加依赖有直接下载jar包和使用cradle的dependencies方法，我们这里直接使用dependencies方法

### 添加gradle依赖

在你的项目的build.grade文件中添加

```
 provided 'org.projectlombok:lombok:1.12.6'
```

至于为什么是provided而不是compile，因为这个框架是在将java编译为class前处理代码了，意思是在生成的class文件中已经生成了getter和setter，所以这个依赖是我们在编译的时候使用，不需要打包到apk中

## 安装Lombok插件

虽然我们添加了依赖，但是Android Studio他知道怎么处理这个文件吗，肯定是不知道啦，所以我们的安装一个插件来告诉他怎么处理

Preferences > Plugins > Browse repositories

在输入框内输入combo，可看到已经搜索出来了这个插件，我们点击旁边的安装，安装完成后重启插件我们就安装完毕了，它的使用使用说明可以查看[插件主页](https://github.com/mplushnikov/lombok-intellij-plugin)

[![](http://7qnc6h.com1.z0.glb.clouddn.com/lombok_plugins.png)](http://7qnc6h.com1.z0.glb.clouddn.com/lombok_plugins.png)

现在插件虽然安装完了，但是Android Studio他怎么知道什么时候来使用这个插件呢，他是不是有个开关什么的，没错！你猜对了

## 开启项目的Annotation process

首先我们打开项目的设置，要强调的是项目的，而不是工具全局的设置，如下图

[![](http://7qnc6h.com1.z0.glb.clouddn.com/project_pre.png)](http://7qnc6h.com1.z0.glb.clouddn.com/project_pre.png)

打开后按照下图勾选Enable annotation processing

[![](http://7qnc6h.com1.z0.glb.clouddn.com/Snip20160619_2.png)](http://7qnc6h.com1.z0.glb.clouddn.com/Snip20160619_2.png)

## 配置文件 lombok.config
然后在Module根目录下新建一个lombok.config配置文件，加入两行配置
>lombok.anyConstructor.suppressConstructorProperties = true
lombok.addGeneratedAnnotation = false

两个配置的作用是禁用掉一些在android上不存在的类，这些类在标准java中才存在。

到这里为止，工具和基本环境我们基本配置完了，接下来我们需要创建一个项目来测试

## 一些常用注解

### @Getter and @Setter

可以很直观的从名字看出这个两个是分别用来生成Getter和Setter方法的


```
 public class User {
 	@Getter @Setter private boolean employed = true;
 	@Setter(AccessLevel.PROTECTED) private String name;
 }
```

相当于

```
 public class User {

 	private boolean employed = true;
 	private String name;

 	public User() {
 	}

 	public boolean isEmployed() {
 		return this.employed;
 	}

 	public void setEmployed(boolean employed) {
	 	this.employed = employed;
 	}

	 protected void setName(String name) {
 		this.name = name;
 	}
 }

```


到这里，大家肯定要问了，你说相当就等啊，或者上面相等的代码是怎么来的呢，其他我们可以直接反编译生成的class文件查看，打开Android Studio的class目录下的User.class文件，可以看到

[![](http://7qnc6h.com1.z0.glb.clouddn.com/Snip20160619_3.png)](http://7qnc6h.com1.z0.glb.clouddn.com/Snip20160619_3.png)

同样下面的实例代码你可以这样查看，当然我们也可用通过Android Studio的

[![](http://7qnc6h.com1.z0.glb.clouddn.com/Snip20160619_4.png)](http://7qnc6h.com1.z0.glb.clouddn.com/Snip20160619_4.png)

从上图我们发现我们没有在源代码写一些Getter方法，但是从Structure窗口看到这些方法已经自动生成了，是不是很神奇

### @NonNull

提供一个参数的非空判断

```
 @Getter @Setter @NonNull
 private List<String> members;
```

等同于

```
 @NonNull
 private List<String> members;

 @NonNull
 public List<String> getMembers() {
 	return this.members;
 }

 public void setMembers(@NonNull List<String> members) {
 	if(members == null) {
 		throw new NullPointerException("members");
 	} else {
 		this.members = members;
 	}
 }

```

### @ToString

```
@ToString(exclude="name")
public class User {
    @Getter
    @Setter
    private boolean employed = true;
    @Setter(AccessLevel.PROTECTED)
    private String name;
    @Getter
    @Setter
    @NonNull
    private List<String> members;
}

```

等同于

```
public String toString() {
    return "User(employed=" + this.isEmployed() + ", members=" + this.getMembers() + ")";
}

```

### @EqualsAndHashCode

```
@EqualsAndHashCode(exclude={"name"})
public class User {
    @Getter
    @Setter
    private boolean employed = true;
    @Setter(AccessLevel.PROTECTED)
    private String name;
    @Getter
    @Setter
    @NonNull
    private List<String> members;
}

```

等同于

```
 public boolean equals(Object o) {
    if(o == this) {
        return true;
    } else if(!(o instanceof User)) {
        return false;
    } else {
        User other = (User)o;
        if(!other.canEqual(this)) {
            return false;
        } else if(this.isEmployed() != other.isEmployed()) {
            return false;
        } else {
            List this$members = this.getMembers();
            List other$members = other.getMembers();
            if(this$members == null) {
                if(other$members != null) {
                    return false;
                }
            } else if(!this$members.equals(other$members)) {
                return false;
            }
            return true;
        }
    }
}
public boolean canEqual(Object other) {
    return other instanceof User;
}
public int hashCode() {
    boolean PRIME = true;
    byte result = 1;
    int result1 = result * 59 + (this.isEmployed()?79:97);
    List $members = this.getMembers();
    result1 = result1 * 59 + ($members == null?0:$members.hashCode());
    return result1;
}

```

### @Data

这个注解相当于同时使用@ToString, @EqualsAndHashCode, @Getter和@Setter

```
@Data
public class User {
    private boolean employed = true;
    private String name;
    private List<String> members;
}

```

等同于

```
 public class User {
    private boolean employed = true;
    private String name;
    private List<String> members;
    public User() {
    }
    public boolean isEmployed() {
        return this.employed;
    }
    public String getName() {
        return this.name;
    }
    public List<String> getMembers() {
        return this.members;
    }
    public void setEmployed(boolean employed) {
        this.employed = employed;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void setMembers(List<String> members) {
        this.members = members;
    }
    public boolean equals(Object o) {
        if(o == this) {
            return true;
        } else if(!(o instanceof User)) {
            return false;
        } else {
            User other = (User)o;
            if(!other.canEqual(this)) {
                return false;
            } else if(this.isEmployed() != other.isEmployed()) {
                return false;
            } else {
                String this$name = this.getName();
                String other$name = other.getName();
                if(this$name == null) {
                    if(other$name != null) {
                        return false;
                    }
                } else if(!this$name.equals(other$name)) {
                    return false;
                }
                List this$members = this.getMembers();
                List other$members = other.getMembers();
                if(this$members == null) {
                    if(other$members != null) {
                        return false;
                    }
                } else if(!this$members.equals(other$members)) {
                    return false;
                }
                return true;
            }
        }
    }
    public boolean canEqual(Object other) {
        return other instanceof User;
    }
    public int hashCode() {
        boolean PRIME = true;
        byte result = 1;
        int result1 = result * 59 + (this.isEmployed()?79:97);
        String $name = this.getName();
        result1 = result1 * 59 + ($name == null?0:$name.hashCode());
        List $members = this.getMembers();
        result1 = result1 * 59 + ($members == null?0:$members.hashCode());
        return result1;
    }
    public String toString() {
        return "User(employed=" + this.isEmployed() + ", name=" + this.getName() + ", members=" + this.getMembers() + ")";
    }
}

```

### @Cleanup

他可以帮我们在需要释放的资源位置自动加上释放代码

```
 public void testCleanUp() {
    try {
        @Cleanup ByteArrayOutputStream baos = new ByteArrayOutputStream();
        baos.write(new byte[] {'Y','e','s'});
        System.out.println(baos.toString());
    } catch (IOException e) {
        e.printStackTrace();
    }
}

```

等同于

```
 public void testCleanUp() {
    try {
        ByteArrayOutputStream e = new ByteArrayOutputStream();
        try {
            e.write(new byte[]{(byte)89, (byte)101, (byte)115});
            System.out.println(e.toString());
        } finally {
            if(Collections.singletonList(e).get(0) != null) {
                e.close();
            }
        }
    } catch (IOException var6) {
        var6.printStackTrace();
    }
}

```

### @Synchronized

可以帮我们在方法上添加同步代码块

```
 public class TestSync {
    private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");
    @Synchronized
    public String synchronizedFormat(Date date) {
        return format.format(date);
    }
}

```

等同于

```
 public class TestSync {
    private final Object $lock = new Object[0];
    private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");
    public TestSync() {
    }
    public String synchronizedFormat(Date date) {
        Object var2 = this.$lock;
        synchronized(this.$lock) {
            return this.format.format(date);
        }
    }
}

```

基本的使用就介绍到这里，下面一份完整代码
User.java

```
package com.woblog.testlombok;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.List;
import lombok.AccessLevel;
import lombok.Cleanup;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NonNull;
import lombok.Setter;
import lombok.SneakyThrows;
import lombok.Synchronized;
import lombok.ToString;
/**
 * Created by renpingqing on 16/6/18.
 */
@Data
@ToString(callSuper = true)
/**
 * 这里如果不写false,会报错,
 * Error:(21, 1) 警告: Generating equals/hashCode implementation but without a call to superclass,
 * even though this class does not extend java.lang.Object. If this is intentional,
 * add '@EqualsAndHashCode(callSuper=false)' to your type.
 */
@EqualsAndHashCode(callSuper = false)
public class User extends BasePOJO {
    private final Object lockObj = new Object();
    /**
     * userid只生成getUserId
     */
    private
    @Getter
    String userId;
    private String username;
    private int gender;
    private float price;
    @Setter(AccessLevel.PROTECTED)
    private boolean vip = false; //vip,或者isVip都会生成isVip
    private List<String> members;
    /**
     * name不能为空
     *
     * @param name
     * @return
     */
    private String sayHello(@NonNull String name) {
        return String.format("hi %s", name);
    }
    @SneakyThrows(IOException.class)
    public void closeabale() {
        @Cleanup InputStream is = new ByteArrayInputStream("hello world".getBytes());
        System.out.println(is.available());
    }
    @Synchronized("lockObj")
    public void lockMethod() {
        System.out.println("test lock method");
    }
}
```

他生成的代码如下：

```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//
package com.woblog.testlombok;
import com.woblog.testlombok.BasePOJO;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.util.Collections;
import java.util.List;
import lombok.NonNull;
public class User extends BasePOJO {
    private final Object lockObj = new Object();
    private String userId;
    private String username;
    private int gender;
    private float price;
    private boolean vip = false;
    private List<String> members;
    private String sayHello(@NonNull String name) {
        if(name == null) {
            throw new NullPointerException("name");
        } else {
            return String.format("hi %s", new Object[]{name});
        }
    }
    public void closeabale() {
        try {
            ByteArrayInputStream $ex = new ByteArrayInputStream("hello world".getBytes());
            try {
                System.out.println($ex.available());
            } finally {
                if(Collections.singletonList($ex).get(0) != null) {
                    $ex.close();
                }
            }
        } catch (IOException var6) {
            throw var6;
        }
    }
    public void lockMethod() {
        Object var1 = this.lockObj;
        synchronized(this.lockObj) {
            System.out.println("test lock method");
        }
    }
    public User() {
    }
    public Object getLockObj() {
        return this.lockObj;
    }
    public String getUsername() {
        return this.username;
    }
    public int getGender() {
        return this.gender;
    }
    public float getPrice() {
        return this.price;
    }
    public boolean isVip() {
        return this.vip;
    }
    public List<String> getMembers() {
        return this.members;
    }
    public void setUserId(String userId) {
        this.userId = userId;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public void setGender(int gender) {
        this.gender = gender;
    }
    public void setPrice(float price) {
        this.price = price;
    }
    public void setMembers(List<String> members) {
        this.members = members;
    }
    public String toString() {
        return "User(super=" + super.toString() + ", lockObj=" + this.getLockObj() + ", userId=" + this.getUserId() + ", username=" + this.getUsername() + ", gender=" + this.getGender() + ", price=" + this.getPrice() + ", vip=" + this.isVip() + ", members=" + this.getMembers() + ")";
    }
    public boolean equals(Object o) {
        if(o == this) {
            return true;
        } else if(!(o instanceof User)) {
            return false;
        } else {
            User other = (User)o;
            if(!other.canEqual(this)) {
                return false;
            } else {
                label71: {
                    Object this$lockObj = this.getLockObj();
                    Object other$lockObj = other.getLockObj();
                    if(this$lockObj == null) {
                        if(other$lockObj == null) {
                            break label71;
                        }
                    } else if(this$lockObj.equals(other$lockObj)) {
                        break label71;
                    }
                    return false;
                }
                String this$userId = this.getUserId();
                String other$userId = other.getUserId();
                if(this$userId == null) {
                    if(other$userId != null) {
                        return false;
                    }
                } else if(!this$userId.equals(other$userId)) {
                    return false;
                }
                label57: {
                    String this$username = this.getUsername();
                    String other$username = other.getUsername();
                    if(this$username == null) {
                        if(other$username == null) {
                            break label57;
                        }
                    } else if(this$username.equals(other$username)) {
                        break label57;
                    }
                    return false;
                }
                if(this.getGender() != other.getGender()) {
                    return false;
                } else if(Float.compare(this.getPrice(), other.getPrice()) != 0) {
                    return false;
                } else if(this.isVip() != other.isVip()) {
                    return false;
                } else {
                    List this$members = this.getMembers();
                    List other$members = other.getMembers();
                    if(this$members == null) {
                        if(other$members != null) {
                            return false;
                        }
                    } else if(!this$members.equals(other$members)) {
                        return false;
                    }
                    return true;
                }
            }
        }
    }
    public boolean canEqual(Object other) {
        return other instanceof User;
    }
    public int hashCode() {
        boolean PRIME = true;
        byte result = 1;
        Object $lockObj = this.getLockObj();
        int result1 = result * 59 + ($lockObj == null?0:$lockObj.hashCode());
        String $userId = this.getUserId();
        result1 = result1 * 59 + ($userId == null?0:$userId.hashCode());
        String $username = this.getUsername();
        result1 = result1 * 59 + ($username == null?0:$username.hashCode());
        result1 = result1 * 59 + this.getGender();
        result1 = result1 * 59 + Float.floatToIntBits(this.getPrice());
        result1 = result1 * 59 + (this.isVip()?79:97);
        List $members = this.getMembers();
        result1 = result1 * 59 + ($members == null?0:$members.hashCode());
        return result1;
    }
    public String getUserId() {
        return this.userId;
    }
    protected void setVip(boolean vip) {
        this.vip = vip;
    }
}

```

可以看到代码量差不多缩减了三倍，但是他也有一些缺点，比如：

大大降低了源代码文件的可读性和完整性，降低了阅读源代码的舒适度。
