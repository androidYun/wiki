 
 ```java
 public static void main(String[] args) {
        testEquals();
    }
    
    /**
     * == 比较的是是否是同一个对象，也相当于比较的是否是同一个地址
     * <p>
     * equals 比较的是值是否相等
     */
    
    private static void testEquals() {
        //首先创建 一个值为 12 的对象，然后存放在常量池里面，相当于a引用值为 12 的对象
        String a = "12";
        //先从常量池里面查找是否有值为 12 的对象  如果 有就，b引用值为 12 的对象，相当于  a和b引用的是一个对象
        String b = "12";

        //由此得出结论 a b 指向同一个对象  a==b  等于true  ，因为值也相等 a.equals(b) 也等于 true

        System.out.println("a==b的值是" + (a == b));

        System.out.println("a.equals(b)的值是" + (a.equals(b)));

        //new 一个值为ab的对象 存放在堆里面，然后c引用这个对象
        String c = new String("ab");//true
        //new 一个值为ab的对象 存放在堆里面，然后d引用这个对象
        String d = new String("ab");//true
        //由此可说 c 和d 不是一个对象 那么 c==d 等于false，因为 c和d的值相等 那么 equals 是先比较是否是同一个对象，如果是一个对象就相等，不是同一个对象就比较值 那么 c.equals(d) 等于 true

        System.out.println("c==d的值是" + (c == d)); //false
        System.out.println("c.equals的值是" + (c.equals(d)));//true

        //因为String 是final 是不可改变的，那么这个字符串 由5个字符组成的，也相当于创建了五个字符串，那么常量池里面可能多了5个字符常量
        String e = "a" + "b" + "c" + "d";
        //因为上面已经创建 值为 abcd 的对象，那么 f只需要引用这个对象就行了 ，那么 e和f引用的是同一个对象,那么 == 和equals 都是相等的
        String f = "abcd";
        System.out.println("e==f的值是" + (e == f)); //true
        System.out.println("e.equals(f)的值是" + (e.equals(f)));//true

        //因为String 是final 是不可改变的，那么这个字符串 由5个字符组成的，也相当于创建了五个字符串，那么常量池里面可能多了5个字符常量
        String g = "a" + "b" + "c" + "d";
        //创建一个值为 abcd的对象 然后存放在堆里面，那么 g和i存放地址不一样，所以==不相等 equals 是相等的 因为 他们值 一样
        String i = new String("abcd");

        System.out.println("g==i的值是" + (g == i)); //false
        System.out.println("g.equals(i)的值是" + (g.equals(i)));//true
    }
    
    ```
