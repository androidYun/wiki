Stask和Queue见解

## 作者|李桂云

定义

stask和Queue项目源码经常用到的东西，平常写代码很少用到，但是androidApi实现原理里面用到的很多，比如启动Activity保存方式，handle  消息保存的方式，

### Stack 解释

stack 中文名是栈，可以保存基本数据和泛型对象，他的保存方式是 后进先出原则，就像向一本盒子里面装书，一次只能拿出一本，所以你只能先取出先放进去的，才能取出下面的，

### Stack 实现方式

Stack 是由基本数据 数组实现的,也就是说我们往Stack里面 增删改查的数据，都是通过数组保存的,数组的好处和弊端，

我们都知道数组，添加和查找 速度是比较可以的，但是删除插入速度不太友好，所以Stack 和数组相辅相成，毕竟基本方式是通过数组实现的

### Stack 添加

Stack 添加一个元素，就是在保存的数组里面添加一个元素，

第一步：判断数组的容量是否够添加一个元素

第二步：如果数组容量不足，那就扩容，根据数组扩容的方法，就是添加到数组目前容量的二倍

第三步：在数组目前保存数据基础上加一等于N，把当前需要添加的值，赋值给增加数组N位置

代码实现原理

```java
//判断当前数组容量和目前Stack 添加数据界面的位置
//如果等于，说明数组容量使用完了！需要扩容 否则把需要添加的数据赋值给当前位置+1
    if (elementCount == elementData.length) {
    //扩容

            growByOne();
        }
        //把需要添加的数据 赋值给 目前位置+1

        elementData[elementCount++] = object;
        modCount++;
        //扩容
 private void growByOne() {
 //默认是扩容量为0

        int adding = 0;
        //表示目前增容量 这个判断是 目前第一次扩容

        if (capacityIncrement <= 0) {
        //判断是否是空栈，如果是空栈，曾增容量是1
        //这句代码看着简单实际很重要 首先先把数据的容量赋值给 adding 也就是灯火扩容量的参数，如果目前数据等于0 
        //扩容量是1 否则 扩容量是当前数组的长度        

            if ((adding = elementData.length) == 0) {
                adding = 1;
            }
        } else {
        //如果capacityIncrement 大于0 则每次扩容都是固定扩容
                adding = capacityIncrement;
        }
        //创建一个增加容量的新数组，
        E[] newData = newElementArray(elementData.length + adding);
        //把目前数组里面的内容赋值到新数组里面

        System.arraycopy(elementData, 0, newData, 0, elementCount);
        //然后把新创建的数组 赋值给之前的数组

        elementData = newData;
    }
    
//解释capacityIncrement的增长原理

//首先是构造方面里面可以赋值，否则，扩容量增值 否则为0
    public Vector(int capacity, int capacityIncrement) {
        this.capacityIncrement = capacityIncrement;
    }
   
```

### Stack 出栈

当初Stack 的原则是后进先出，出栈就是 删除数组里面数组最后一个，数组删除数据很简单，就把最后一个元素删除 赋值为null，stack里面只提供了 删除某个元素，并不是 只删除最后一个，我们分析一下，删除逻辑

第一步：先获取这个元素的位置，如果不存在就不删除

第二步：如果该元素存在，根据位置 先判断这个位置是否合法，是否小于数组的长度，否则报异常

第三步:判断删除这个元素是否是最后一个，如果是最后一个，直接给最后一个赋值给null

第四步:如果不是最后一个那就 把当前要删除的值，覆盖掉

```java
 public synchronized void removeElementAt(int location) {
 //判断当前要删除的数据位置是否合法

        if (location >= 0 && location < elementCount) {
        //elementCount 是数组的长度 最后一个元素的位置是 elementCount-1

            elementCount--;
            //下面两行代码判断是否是最后一个元素

            int size = elementCount - location;
            //如果不是最后一个 当前数据位置+1 之后的数据 向前移动一位

            if (size > 0) {
                System.arraycopy(elementData, location + 1, elementData,
                        location, size);
            }
     //因为已经删除数据 所以把最后一位置空

            elementData[elementCount] = null;
            modCount++;
        } else {
            throw arrayIndexOutOfBoundsException(location, elementCount);
        }
    }
```



### 改和查找都很简单不做分析
































