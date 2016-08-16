---
title: 使用kotlin打造的sharepreference
date: 2016/04/03
tags: 
- android
- kotlin
- sharepreference
categories:
- android
- DataSave
- SharePreference
---
前段时间接触到了**kotlin**这个神奇的语言,于是就迫不及待的将其应用到了我的应用当中,
真正使用过后对其是爱不释手.如果还不知道**kotlin**,[传送门](http://kotlinlang.cn/)在这里.
平时我们对数据的存储方式无非就是数据库,`Sharepreference`,文件存储等这几种,
这几个各有各的优点各有各的用途,对于他们的具体特性及用途我在这里就不说了,
这篇文章主要就是记录一下我使用**kotlin**对`Sharepreference`的一个封装.
<!--more-->
## 效果
对于代码放在下边展示,这里先展示使用的方法
``` kotlin
private var string: String by Preference(ctx, "key", "default value")
```
然后在下面的代码中可以直接使用`string`这个属性,
在使用的时候对其赋值就直接写入到`sharepreference`文件中了,使用起来是不是很方便呢.
## 实现
对于`sharepreference`这样的实现方式主要是应用了**kotlin**的委托属性.
如果想了解**kotlin**的委托属性,这里放上 [传送门](http://kotlindoc.com/ClassesAndObjects/DelegationProperties.html).
下边就直接上代码了.
``` kotlin 
import android.content.Context
import kotlin.properties.ReadWriteProperty
import kotlin.reflect.KProperty
/**
 * 描述:简单的应用属性配置存储文件
 */
class Preference<T>(val context: Context, val key: String, val defVal: T) :
        ReadWriteProperty<Any?, T> {
    val prefs by lazy { context.getSharedPreferences(C.CONFIG_NAME, Context.MODE_PRIVATE) }
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return find(key, defVal)
    }
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        put(key, value)
    }
    /**
    * 查找key对应的值,根据默认值def的类型输出
    */
    private fun <U> find(key: String, def: U): U = with(prefs) {
        val res: Any = when (def) {
            is Boolean -> getBoolean(key, def)
            is String -> getString(key, def)
            is Float -> getFloat(key, def)
            is Int -> getInt(key, def)
            is Long -> getLong(key, def)
            is Set<*> -> getStringSet(key, def as Set<String>)
            else -> {
                throw IllegalArgumentException("输入的格式有误")
            }
        }
        res as U
    }
    /**
    * 放入一个键值对
    */
    private fun <U> put(key: String, value: U) = with(prefs.edit()) {
        when (value) {
            is Boolean -> putBoolean(key, value)
            is String -> putString(key, value)
            is Float -> putFloat(key, value)
            is Int -> putInt(key, value)
            is Long -> putLong(key, value)
            is Set<*> -> putStringSet(key, value as Set<String>)
            else -> throw IllegalArgumentException("输入的格式有误")
        }.apply()
    }
}
```
如上的代码就是整个封装过后的sharepreference类了,代码也不是很复杂,
上面的代码中用到了[泛型](http://my.oschina.net/jiemachina/blog/201507),
使用泛型可以使我们的代码更加的简单,不然上述代码就要复杂的多了.
## 缺陷
当然讲完了好处,也要说明他的缺陷了,毕竟没有绝对完美的东西,只能尽量的完善
- 首先这样的写法在kotlin中调用时没有问题的,但是在java中调用,恕我智商低,
还没有找到一个好的调用方法.
- 其定义只能放在类变量里边,这样造成的问题就是在fragment中,
content会出现空指针现象
- 其他问题,以后发现了再说☺