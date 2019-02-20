# Android 状态模式实践

### 什么是状态模式

在不同状态下，具有不同的行为

### 使用场景

比如 我们发货方式有两种方式 快递，物流，使用快递 都需要 打包，计算费用，因为使用快递比较小，物流往往比较大的，所以打包方式不一样，计算费用也不一样，需要这种情况我们都会 用 if else if() 下面我随便写一下代码

```java
  
  //我们都是先判断湿 那种方式 然后打包，计算金额 如果快递有很多种费用都不一样，就会一直 if else if 下面我
  //用项目实战中用到的状态模式 来表述状态模式是什么东西
  if (status.equals("快递")) {
            packageExpressDelivery();
            calculateAmountExpressDelivery();
        } else if (status.equals("物流")) {
            packageLogistics();
            calculateLogisticsAmount();
        }
         /**
     * 打包物流
     */
    private void packageLogistics() {

    }

    /**
     * 计算费用
     */
    private double calculateLogisticsAmount() {
        return 0;
    }

    /**
     * 打包快递
     */
    private void packageExpressDelivery() {

    }

    /**
     * 计算快递费用
     */
    private double calculateAmountExpressDelivery() {
        return 0;
    }

```

### 项目实战用的状态模式

##### 一，项目需求

选择充值金额方式，目前有两种，1，快捷充值（速度快，手续费比较高，转账到任何账户里面）2，转账充值（速度满，手续费低 转账到银行卡里面） 以后会添加 平安，富有，支付宝，微信 等充值方式

##### 二，为了以后充值方式的扩展，到达改动小，添加快

确定需求：1，选择不同的支付方式跳转到不同界面去充值

2，手续费计算方式不一样

3，因为手续费不一样 到账金额也不一样

##### 三，定义接口

```java
public interface RechargeState {
//跳转到不同界面
    void jumpActivity(Context context);
//计算到账金额
    BigDecimal getArrivalAmount(BigDecimal rechargeAmount,BigDecimal serviceFee,BigDecimal minServiceCharge);
//计算服务费用
    BigDecimal getServiceAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge);
}

```

##### 四 不同的充值方式实现上面接口，并且业务不同实现不同业务逻辑

```java
public class QuitRechargeState implements RechargeState {

    @Override
    public void jumpActivity(Context context) {
    //跳转到快捷充值界面去充值
        context.startActivity(new Intent(context, QuitRechargeActivity.class));
    }

    @Override
    public BigDecimal getArrivalAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
    //金额-金额*费率就是 到账金额
        return rechargeAmount.subtract(getRechargeFee(rechargeAmount, serviceFee));
    }

    @Override
    public BigDecimal getServiceAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
//如果手续费小于最小金额  就去最小金额 否则 取 金额*费率
        if (getRechargeFee(rechargeAmount, serviceFee).compareTo(minServiceCharge) < 0) {
            return minServiceCharge;
        } else {
            return getRechargeFee(rechargeAmount, serviceFee);
        }
    }
/*
*计算手续费
*/
    private BigDecimal getRechargeFee(BigDecimal rechargeAmount, BigDecimal serviceFee) {
        return rechargeAmount.multiply(serviceFee).setScale(2, RoundingMode.HALF_UP);
    }
}

```

```java
public class TransferRechargeState implements RechargeState {

    @Override
    public void jumpActivity(Context context) {
    //跳转到转账金额
        context.startActivity(new Intent(context, TransferRechargeActivity.class));
    }

    @Override
    public BigDecimal getArrivalAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
    //金额-金额*手续费-0.1  0.1 元是 短信通知费用
        return rechargeAmount.subtract(getRechargeFee(rechargeAmount, serviceFee))-2;
    }

    @Override
    public BigDecimal getServiceAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
    //金额*费率 等于 手续费
        return getRechargeFee(rechargeAmount, serviceFee);
    }

    private BigDecimal getRechargeFee(BigDecimal rechargeAmount, BigDecimal serviceFee) {
        return rechargeAmount.multiply(serviceFee).setScale(2, RoundingMode.HALF_UP);
    }
}

```

```java
public class RechargeController   {

    private RechargeState rechargeState;
    
//设置当前模式是快捷充值
    public void setQuitRecharge() {
        rechargeState = new QuitRechargeState();
    }
//设置当前模式为转账充值
    public void setTransferRecharge() {
        rechargeState = new TransferRechargeState();
    }

  
    public void jumpActivity(Context context) {
        if(rechargeState==null){
            new Throwable("please set state");
        }
        rechargeState.jumpActivity(context);
    }

   
    public BigDecimal calculateArrivalAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
        if(rechargeState==null){
            new Throwable("please set state");
        }
        return rechargeState.getArrivalAmount(rechargeAmount, serviceFee, minServiceCharge);
    }

   
    public BigDecimal calculateServiceAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
        if(rechargeState==null){
            new Throwable("please set state");
        }
        return rechargeState.getServiceAmount(rechargeAmount, serviceFee, minServiceCharge);
    }
}

```

上面 代码已经写好了，下面我门看看怎么用

```java
  //创建充值对象
  final RechargeController rechargeController = new RechargeController();
  //设置转账充值
        btn_transfer.setOnClickListener(new View.OnClickListener() {
                                            @Override
                                            public void onClick(View v) {
                                                rechargeController.setTransferRecharge();
                                            }
                                        }
        );
        //设置快捷充值
        btn_quit.setOnClickListener(new View.OnClickListener() {
                                        @Override
                                        public void onClick(View v) {
                                            rechargeController.setTransferRecharge();
                                        }
                                    }
        );
        //跳转到Activity
        rechargeController.jumpActivity(this);
        //计算到账金额
        rechargeController.calculateArrivalAmount(new BigDecimal(100), new BigDecimal(0.0038), new BigDecimal(2));
        //计算手续费
        rechargeController.calculateServiceAmount(new BigDecimal(100), new BigDecimal(0.0038), new BigDecimal(2));

```

看到上面代码 是不是很清晰明了，光清晰 还不算，如果续费变更 需要 添加 支付宝充值，看看我怎么添加 首先创建一个对象并且实现 我们定义的接口 看下面的代码

```java
public class AlipayRechargeState implements RechargeState {
    @Override
    public void jumpActivity(Context context) {
        context.startActivity(new Intent(context, AlipayRechargeActivity.class));
    }

    @Override
    public BigDecimal getArrivalAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
        return rechargeAmount.subtract(getRechargeFee(rechargeAmount, serviceFee));
    }

    @Override
    public BigDecimal getServiceAmount(BigDecimal rechargeAmount, BigDecimal serviceFee, BigDecimal minServiceCharge) {
        return getRechargeFee(rechargeAmount, serviceFee);
    }

    private BigDecimal getRechargeFee(BigDecimal rechargeAmount, BigDecimal serviceFee) {
        return rechargeAmount.multiply(serviceFee).setScale(2, RoundingMode.HALF_UP);
    }
}
```

然后我们修改 RechargeController 添加一个设置支付宝充值模式

```java
//设置当前模式为转账充值  把这句到吗添加到  RechargeController
 public void setAlipayRecharge() {
 rechargeState = new AlipayRechargeState();
 }
```

是不是很简单 原来的代码逻辑，一句也没动，只需要 在控制添加 支付方式就ok 具体显示教给自己的类

### 状态模式优点

每个状态是在一个子类里面，业务逻辑都在自己的子类里面实现，易于扩张和维护

结构清晰，避免过多的if else if语句 以及状态判断过多，易于混乱，

### 状态模式缺点

导致子类过多
