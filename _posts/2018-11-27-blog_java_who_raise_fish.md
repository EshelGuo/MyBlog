---
layout: post
title: 使用Java程序来解答谁养鱼的问题
description: 
comments: true
tags: [sample post]
image:
  background: abstract-1.jpg
---
爱因斯坦谁养鱼问题题目如下:

	/**
	 * 1 有五栋五种颜色的房子
	 * 2 每一位房子的主人国籍都不同
	 * 3 这五个人每人只喝一种饮料，只抽一种牌子的香烟，只养一种宠物
	 * 4 没有人有相同的宠物，抽相同牌子的香烟，喝相同的饮料
	 *
	 * 提示：
	 * 1 英国人住在红房子里
	 * 2 瑞典人养了一条狗
	 * 3 丹麦人喝茶
	 * 9 挪威人住第一间房子
	 * 13 德国人抽 PRINCE 烟
	 * 7 黄房子主人抽 DUNHILL 烟
	 *
	 * 14 挪威人住在蓝房子旁边
	 * 11 养马人住在 DUNHILL 烟的人旁边
	 *
	 * 4 绿房子在白房子左边
	 * 5 绿房子主人喝咖啡
	 *
	 *
	 * 6 抽 PALL MALL 烟的人养了一只鸟
	 * 8 住在中间那间房子的人喝牛奶
	 * 10 抽混合烟的人住在养猫人的旁边
	 * 12 抽 BLUE MASTER 烟的人喝啤酒
	 * 15 抽混合烟的人的邻居喝矿泉水
	 * 问题: 谁养鱼?
	 */

根据这个问题, 要想通过程序得到答案, 首先需要建立数据模型:

```java

	enum Home{
	    红房子, 绿房子, 白房子, 黄房子, 蓝房子
	}
	
	enum People{
	    瑞典人, 丹麦人, 挪威人, 德国人, 英国人
	}
	
	enum Animal{
	    DOG, 鸟, 猫, 马, 鱼
	}
	
	enum Water{
	    咖啡, 牛奶, 啤酒, 矿泉水, 茶
	}
	
	enum Smoke{
	    pallmall烟, dunhill烟, 混合烟, bluemaster烟, prince烟
	}

```

根据某个人住的房子, 养的宠物, 抽的烟喝的饮料, 建立数据模型 E:

```java

	class E{
	    Home mHome;
	    People mPeople;
	    Animal mAnimal;
	    Water mWater;
	    Smoke mSmoke;
	
	    public E(Home home, People people, Animal animal, Water water, Smoke smoke) {
	        mHome = home;
	        mPeople = people;
	        mAnimal = animal;
	        mWater = water;
	        mSmoke = smoke;
	    }
	
	    @Override
	    public String toString() {
	        return "E{" +
	                "mHome=" + mHome +
	                ", mPeople=" + mPeople +
	                ", mAnimal=" + mAnimal +
	                ", mWater=" + mWater +
	                ", mSmoke=" + mSmoke +
	                '}';
	    }
	}

```

然后根据第1 2 3 9 13 条提示, 得出最初的数据集(目前没有先后顺序):

```java

	sEs = new E[5];
	sE1 = new E(null, People.挪威人,null, null, null);
	sE2 = new E(Home.红房子, People.英国人,null, null, null);
	sE3 = new E(null, People.德国人,null, null, Smoke.prince烟);
	sE4 = new E(null, People.瑞典人,Animal.DOG, null, null);
	sE5 = new E(null, People.丹麦人,null, Water.茶, null);

```

思考一个问题, 挪威人住什么房子?

 - 由 1 英国人住在红房子里 得知挪威人不住 红房子
 - 由 14 挪威人住在蓝房子旁边 得知挪威人不住 蓝房子

即挪威人可能住[黄房子, 绿房子, 白房子]

根据 9 得知挪威人在第一栋房子

根据 14 得知第二栋房子是蓝房子

 - 假设挪威人住绿房子, 由 4 绿房子在白房子左边 得知第二栋房子是白房子, 与 第二栋房子是蓝房子冲突, 不成立
 - 假设挪威人住白房子, 则 9 得知挪威人在第一栋房子 与 4 冲突, 故不成立

初步结论: 挪威人住黄房子.

根据 7 黄房子主人抽 DUNHILL 烟 , 以及以上结论, 重新更新数据集(不分先后顺序):

```java

	sEs = new E[5];
    sE1 = new E(Home.黄房子, People.挪威人,null, null, Smoke.dunhill烟);
    sE2 = new E(Home.红房子, People.英国人,null, null, null);
    sE3 = new E(null, People.德国人,null, null, Smoke.prince烟);
    sE4 = new E(null, People.瑞典人,Animal.DOG, null, null);
    sE5 = new E(null, People.丹麦人,null, Water.茶, null);

```

null 即未知

定义数组 sEs 来按照顺序存储5个 E 对象, 由9得知 `sEs[0] = sE1;`

由14得知sEs[1] 为蓝房子, 那么问题来了, 谁住蓝房子? 
定义一个方法 `assertBlueHome()` 来推断谁住蓝房子:

 - 根据已知数据集排除 sE1 和 sE2, 因为它们的房子是已知的
 - 根据 11 养马人住在 DUNHILL 烟的人旁边 得知蓝房子养马
 - 根据已知数据集排除 sE4, 因为他养的动物是已知的
 - 故 assertBlueHome() 有两个可能结果: sE3 和 sE5

接着需要遍历这个结果, 对可能结果进行赋值:

	sEs[1] = e;
	e.mHome = Home.蓝房子;
	e.mAnimal = Animal.马;

在此基础上对其他结果进行尝试, 最后要将尝试的赋值还原回来, 继续下一轮尝试:

	sEs[1] = e;
	e.mHome = Home.蓝房子;
	e.mAnimal = Animal.马;
	tryBlueHome();
	e.mHome = null;
    e.mAnimal = null;
    sEs[1] = null;

故每一轮尝试都要赋值, 尝试, 还原, 所以我定义接口 Try 来做这件事:

```java

	interface Try{
	    void prepareTry(E e);
	    boolean try_(E e);
	    void tryOver(E e);
	}

```

实现 Try 接口, 在 prepareTry 中进行赋值, try_ 中尝试, 返回 true 则结果成立, 否则不成立, 在 tryOver 中将之前的赋值还原.

定义函数tryE()来执行Try:

```java

    private static boolean tryE(E e,Try try_){
        count++;//统计代码, 不用管
        if(try_ == null || e == null)
            return false;
        try_.prepareTry(e);
        boolean success = try_.try_(e);
        if(!success)
            try_.tryOver(e);//未成功才需要还原, 尝试成功不需要还原
        return success;//返回成功与否
    }

```

循环尝试蓝房子的结果:

```java

	sEs[0] = e1;
    for (E e : assertBlueHome()) {
        boolean success = tryE(e, new Try() {
            @Override
            public void prepareTry(E e) {
                sEs[1] = e;
                e.mHome = Home.蓝房子;
                e.mAnimal = Animal.马;
            }

            @Override
            public boolean try_(E e) {
                return tryBlueHome();
            }

            @Override
            public void tryOver(E e) {
                e.mHome = null;
                e.mAnimal = null;
                sEs[1] = null;
            }
        });
        if(success){
            System.out.println("-------------------------- start -----------------------------");
            System.out.println(Arrays.toString(sEs));
            System.out.println("--------------------------- end ------------------------------");
            System.out.println(count);
        }
    }

```

然后根据已知条件和条件 8 推断中间房子的可能结果(即除去挪威人 蓝房子, 所有 mWater 为空的集合):

```java

	private static boolean tryBlueHome() {
        for (E assertE3 : assertE3Home()) {
            boolean success = tryE(assertE3, new Try() {
                @Override
                public void prepareTry(E e) {
                    sEs[2] = e;
                    e.mWater = Water.牛奶;
                }

                @Override
                public boolean try_(E e) {
                    return tryGreenHome();
                }

                @Override
                public void tryOver(E e) {
                    if (e != null)
                        e.mWater = null;
                    sEs[2] = null;
                }
            });
            if(success)
                return true;
        }
        return false;
    }

```

然后
 
- 根据条件 5 在蓝房子基础上推断绿房子`tryGreenHome()` ,  
- 根据条件 4 在绿房子基础上推断白房子`tryWhiteHome(E greenHome)`
- 根据条件 6 在白房子的基础上推断抽 Pallmall 烟的人 `tryPallmall()`
- 根据条件 12 在Pallmall的基础上推断抽 Bluemaster 烟的人 `tryBluemaster()`
- 在Bluemaster的基础上推断抽 混合烟 烟的人 `tryBlend()`
- 最后只剩下两个条件:
	- 10．抽混合烟的人住在养猫人的旁边
	- 15．抽混合烟的人的邻居喝矿泉水 
- 然后根据这两个条件来检查结果是否成立, 成立返回 true, 不成立返回 false
- 然后将根据返回值一步步将是否成立的结果传递给最外层, 最外层判断成立后打印最终结果. 即谁养鱼的答案

下边贴出完整代码:

&emsp;&emsp;其中 `assertXXX` 都是根据1-15的条件获取可能结果,  `tryXXX`都是根据`assertXXX`尝试赋值, 继续下一个推断.

最后的结论是: 德国人养鱼.
```java
	
	public class Test {
	
	    private static E[] sEs;
	    private static E sE1;
	    private static E sE2;
	    private static E sE3;
	    private static E sE4;
	    private static E sE5;
	    private static ArrayList<E> sAll;
	
	    /**
	     * 1 有五栋五种颜色的房子
	     * 2 每一位房子的主人国籍都不同
	     * 3 这五个人每人只喝一种饮料，只抽一种牌子的香烟，只养一种宠物
	     * 4 没有人有相同的宠物，抽相同牌子的香烟，喝相同的饮料
	     *
	     * 提示：
	     * 1 英国人住在红房子里
	     * 2 瑞典人养了一条狗
	     * 3 丹麦人喝茶
	     * 9 挪威人住第一间房子
	     * 13 德国人抽 PRINCE 烟
	     * 7 黄房子主人抽 DUNHILL 烟
	     *
	     * 14 挪威人住在蓝房子旁边
	     * 11 养马人住在 DUNHILL 烟的人旁边
	     *
	     * 4 绿房子在白房子左边
	     * 5 绿房子主人喝咖啡
	     *
	     *
	     * 6 抽 PALL MALL 烟的人养了一只鸟
	     * 8 住在中间那间房子的人喝牛奶
	     * 10 抽混合烟的人住在养猫人的旁边
	     * 12 抽 BLUE MASTER 烟的人喝啤酒
	     * 15 抽混合烟的人的邻居喝矿泉水
	     * 问题: 谁养鱼?
	     */
	    public static void main(String args[]){
	        sEs = new E[5];
	        sE1 = new E(Home.黄房子, People.挪威人,null, null, Smoke.dunhill烟);
	        sE2 = new E(Home.红房子, People.英国人,null, null, null);
	        sE3 = new E(null, People.德国人,null, null, Smoke.prince烟);
	        sE4 = new E(null, People.瑞典人,Animal.DOG, null, null);
	        sE5 = new E(null, People.丹麦人,null, Water.茶, null);
	
	        sAll = new ArrayList<>(5);
	        sAll.add(sE1);
	        sAll.add(sE2);
	        sAll.add(sE3);
	        sAll.add(sE4);
	        sAll.add(sE5);
	
	        initEs(sE1);
	
	        /*
	        E{mHome=黄房子, mPeople=挪威人, mAnimal=猫, mWater=矿泉水, mSmoke=dunhill烟}
	        E{mHome=蓝房子, mPeople=丹麦人, mAnimal=马, mWater=茶, mSmoke=混合烟}
	        E{mHome=红房子, mPeople=英国人, mAnimal=鸟, mWater=牛奶, mSmoke=pallmall烟}
	        E{mHome=绿房子, mPeople=德国人, mAnimal=null, mWater=咖啡, mSmoke=prince烟}
	        E{mHome=白房子, mPeople=瑞典人, mAnimal=DOG, mWater=啤酒, mSmoke=bluemaster烟}
	*/
	    }
	
	    private static boolean tryE(E e,Try try_){
	        count++;
	        if(try_ == null || e == null)
	            return false;
	        try_.prepareTry(e);
	        boolean success = try_.try_(e);
	        if(!success)
	            try_.tryOver(e);
	        return success;
	    }
	
	    private static void initEs(E e1) {
	        sEs[0] = e1;
	        for (E e : assertBlueHome()) {
	            boolean success = tryE(e, new Try() {
	                @Override
	                public void prepareTry(E e) {
	                    sEs[1] = e;
	                    e.mHome = Home.蓝房子;
	                    e.mAnimal = Animal.马;
	                }
	
	                @Override
	                public boolean try_(E e) {
	                    return tryBlueHome();
	                }
	
	                @Override
	                public void tryOver(E e) {
	                    e.mHome = null;
	                    e.mAnimal = null;
	                    sEs[1] = null;
	                }
	            });
	            if(success){
	                System.out.println("-------------------------- start -----------------------------");
	                System.out.println(Arrays.toString(sEs));
	                System.out.println("--------------------------- end ------------------------------");
	                System.out.println(count);
	            }
	        }
	    }
	
	    private static boolean tryBlueHome() {
	        for (E assertE3 : assertE3Home()) {
	            boolean success = tryE(assertE3, new Try() {
	                @Override
	                public void prepareTry(E e) {
	                    sEs[2] = e;
	                    e.mWater = Water.牛奶;
	                }
	
	                @Override
	                public boolean try_(E e) {
	                    return tryGreenHome();
	                }
	
	                @Override
	                public void tryOver(E e) {
	                    if (e != null)
	                        e.mWater = null;
	                    sEs[2] = null;
	                }
	            });
	            if(success)
	                return true;
	        }
	        return false;
	    }
	
	    private static boolean tryGreenHome() {
	        E[] greenHomes = assertGreenHome();
	        for (E greenHome : greenHomes) {
	            boolean success = tryE(greenHome, new Try() {
	                @Override
	                public void prepareTry(E e) {
	                    e.mHome = Home.绿房子;
	                    e.mWater = Water.咖啡;
	                    sEs[3] = e;
	                }
	
	                @Override
	                public boolean try_(E e) {
	                    return tryWhiteHome(e);
	                }
	
	                @Override
	                public void tryOver(E e) {
	                    e.mHome = null;
	                    e.mWater = null;
	                    sEs[3] = null;
	                }
	            });
	            if(success)
	                return true;
	        }
	        return false;
	    }
	
	    private static boolean tryWhiteHome(final E greenHome) {
	        final E[] whites = assertWhiteHome();
	        if (whites == null || whites.length == 0)
	            return false;
	        for (E white : whites) {
	            boolean success = tryE(white, new Try() {
	                @Override
	                public void prepareTry(E e) {
	                    sEs[getIndexByE(greenHome) + 1] = e;
	                    e.mHome = Home.白房子;
	                }
	
	                @Override
	                public boolean try_(E e) {
	                    return tryPallmall();
	                }
	
	                @Override
	                public void tryOver(E e) {
	                    sEs[getIndexByE(greenHome) + 1] = null;
	                    e.mHome = null;
	                }
	            });
	            if(success)
	                return true;
	        }
	        return false;
	    }
	
	    private static boolean tryPallmall() {
	        E[] pallmallEs = assertPallmall();
	        for (E pallmallE : pallmallEs) {
	            boolean success = tryE(pallmallE, new Try() {
	                @Override
	                public void prepareTry(E pallmallE) {
	                    pallmallE.mSmoke = Smoke.pallmall烟;
	                    pallmallE.mAnimal = Animal.鸟;
	                }
	
	                @Override
	                public boolean try_(E e) {
	                    return tryBluemaster();
	                }
	
	                @Override
	                public void tryOver(E pallmallE) {
	                    pallmallE.mSmoke = null;
	                    pallmallE.mAnimal = null;
	                }
	            });
	            if(success)
	                return true;
	        }
	        return false;
	    }
	
	    private static boolean tryBluemaster() {
	        E[] bluemaster = assertBluemaster();
	        if(arrayIsEmpty(bluemaster))
	            return false;
	        for (E e : bluemaster) {
	            boolean success = tryE(e, new Try() {
	                @Override
	                public void prepareTry(E e) {
	                    e.mSmoke = Smoke.bluemaster烟;
	                    e.mWater = Water.啤酒;
	                }
	
	                @Override
	                public boolean try_(E e) {
	                    return tryBlend();
	                }
	
	                @Override
	                public void tryOver(E e) {
	                    e.mSmoke = null;
	                    e.mWater = null;
	                }
	            });
	            if(success)
	                return true;
	        }
	        return false;
	    }
	
	    static int count;
	    private static boolean tryBlend() {
	        E[] blends = assertBlend();
	        if(arrayIsEmpty(blends))
	            return false;
	        for (final E blend : blends) {
	            boolean success = tryE(blend, new Try() {
	                @Override
	                public void prepareTry(E e) {
	                    e.mSmoke = Smoke.混合烟;
	                }
	
	                @Override
	                public boolean try_(E e) {
	                    //混合烟人的脚标
	                    int blendIndex = getIndexByE(e);
	                    E last = getLast(blendIndex);
	                    E next = getNext(blendIndex);
	                    return checkTheEndResult(last, next);
	                }
	
	                private boolean checkTheEndResult(E last, E next) {
	                    //10．抽混合烟的人住在养猫人的旁边
	                    //15．抽混合烟的人的邻居喝矿泉水
	                    boolean canCattery = false;//养猫
	                    boolean canDrinkWater = false;//喝矿泉水
	                    if(last != null){
	                        canCattery = checkCattery(last);
	                        canDrinkWater = checkDrinkWater(last);
	                    }
	
	                    if(next != null){
	                        if(!canCattery)
	                            canCattery = checkCattery(next);
	                        if(!canDrinkWater)
	                            canDrinkWater = checkDrinkWater(next);
	                    }
	                    if(canCattery && canDrinkWater)
	                        return true;
	
	                    //reset
	                    if(last != null){
	                        if(last.mWater == Water.矿泉水)
	                            last.mWater = null;
	                        if(last.mAnimal == Animal.猫)
	                            last.mAnimal = null;
	                    }
	
	                    if(next != null){
	                        if(next.mWater == Water.矿泉水)
	                            next.mWater = null;
	                        if(next.mAnimal == Animal.猫)
	                            next.mAnimal = null;
	                    }
	                    return false;
	                }
	
	                private boolean checkDrinkWater(E e) {
	                    if(e.mWater == null || e.mWater == Water.矿泉水){
	                        e.mWater = Water.矿泉水;
	                        return true;
	                    }
	                    return false;
	                }
	
	                private boolean checkCattery(E e) {
	                    if(e.mAnimal == null || e.mAnimal == Animal.猫){
	                        e.mAnimal = Animal.猫;
	                        return true;
	                    }
	                    return false;
	                }
	
	                private E getNext(int blendIndex) {
	                    int index = blendIndex + 1;
	                    if(index >= 0 && index < sEs.length)
	                        return sEs[index];
	                    return null;
	                }
	
	                private E getLast(int blendIndex) {
	                    int index = blendIndex - 1;
	                    if(index >= 0 && index < sEs.length)
	                        return sEs[index];
	                    return null;
	                }
	
	                @Override
	                public void tryOver(E e) {
	                    e.mSmoke = null;
	                }
	            });
	            if(success)
	                return true;
	        }
	        return false;
	    }
	
	    private static E[] assertBlend() {
	        ArrayList<E> blend = new ArrayList<>();
	        for (E e : sAll) {
	            if(e.mSmoke != null)
	                continue;
	            blend.add(e);
	        }
	        return blend.toArray(new E[0]);
	    }
	
	    private static E[] assertBluemaster() {
	        //12．抽bluemaster烟的人喝啤酒
	        ArrayList<E> bluemasters = new ArrayList<>();
	        for (E e : sAll) {
	            if(e.mSmoke != null)
	                continue;
	            if(e.mWater != null)
	                continue;
	            bluemasters.add(e);
	        }
	        return bluemasters.toArray(new E[0]);
	    }
	
	    private static E[] assertPallmall() {
	        ArrayList<E> result = new ArrayList<>(5);
	        for (E e : sAll) {
	            if(e.mSmoke != null)
	                continue;
	            if(e.mAnimal != null)
	                continue;
	            result.add(e);
	        }
	        return result.toArray(new E[0]);
	    }
	
	    private static int getIndexByE(E e){
	        for (int i = 0; i < sEs.length; i++) {
	            E e1 = sEs[i];
	            if(e == e1)
	                return i;
	        }
	        return -1;
	    }
	
	    private static E[] assertE3Home() {
	        ArrayList<E> e3 = new ArrayList<>(4);
	        for (int i = 1; i < sAll.size() - 1; i++) {
	            E e = sAll.get(i);
	            if(e.mHome == Home.蓝房子)
	                continue;
	            e3.add(e);
	        }
	        return e3.toArray(new E[0]);
	        //e3 != e5
	        //e3 != e1
	        //e3 != assertBlueHome();
	        //e3 == 绿房子 || e3 == 白房子 || e3 == 红房子
	        //e3 == [e2 e3 e4]
	    }
	
	    /**
	     * 谁住蓝房子?
	     */
	    private static E[] assertBlueHome() {
	        // != e2
	        //蓝房子养马
	        return new E[]{sE3, sE5};
	    }
	
	    private static E[] assertGreenHome(){
	        ArrayList<E> greenE = new ArrayList<>(3);
	        for (int i = 1; i < sAll.size(); i++) {
	            E e = sAll.get(i);
	            if(e.mHome != null)
	                continue;
	            if(e.mWater != null)
	                continue;
	            greenE.add(e);
	        }
	        return greenE.toArray(new E[0]);
	    }
	
	    private static E[] assertWhiteHome(){
	        ArrayList<E> white = new ArrayList<>(2);
	        for (int i = 1; i < sAll.size(); i++) {
	            E e = sAll.get(i);
	            if(e.mHome != null)
	                continue;
	            white.add(e);
	        }
	        return white.toArray(new E[0]);
	    }
	
	    private static boolean arrayIsEmpty(Object[] arrays){
	        return arrays == null || arrays.length == 0;
	    }
	}
	
	class E{
	    Home mHome;
	    People mPeople;
	    Animal mAnimal;
	    Water mWater;
	    Smoke mSmoke;
	
	    public E(Home home, People people, Animal animal, Water water, Smoke smoke) {
	        mHome = home;
	        mPeople = people;
	        mAnimal = animal;
	        mWater = water;
	        mSmoke = smoke;
	    }
	
	    @Override
	    public String toString() {
	        return "E{" +
	                "mHome=" + mHome +
	                ", mPeople=" + mPeople +
	                ", mAnimal=" + mAnimal +
	                ", mWater=" + mWater +
	                ", mSmoke=" + mSmoke +
	                '}';
	    }
	}
	
	enum Home{
	    红房子, 绿房子, 白房子, 黄房子, 蓝房子
	}
	
	enum People{
	    瑞典人, 丹麦人, 挪威人, 德国人, 英国人
	}
	
	enum Animal{
	    DOG, 鸟, 猫, 马, 鱼
	}
	
	enum Water{
	    咖啡, 牛奶, 啤酒, 矿泉水, 茶
	}
	
	enum Smoke{
	    pallmall烟, dunhill烟, 混合烟, bluemaster烟, prince烟
	}
	
	interface Try{
	    void prepareTry(E e);
	    boolean try_(E e);
	    void tryOver(E e);
	}


```