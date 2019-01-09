---
layout: post
title: 枚举
date: 2014-11-20
categories: java
tags: enum effectiveJava
---

这个主要是effective java的笔记，大部分配的是工作中用的例子，部分还是书上的例子

1.  用enum代替int常量，

    >   a.因为没有可以访问的构造器,枚举类型是真正的final    
    >   b.枚举天生就是不可变的,因此所有的域都应该是final的,他们可以是公有的,但最好将他们做成私有的,并提供公有的访问方法.  
    >   c.特定于常量的方法实现,枚举里加接口, 各常量里来实现 ，而不是if-else，switch  
    >   d.策略枚举,

        public enum QmqType {
          CREATE_ORDER(".order", "生单", 99, 99) {
            @Override
            public int getGdp(String site) {
                return ConfigServiceUtil.getGdpAndError(site, QmqType.CREATE_ORDER, ConfigType.GDP);
            }

            @Override
            public int getMaxError(String site) {
                return ConfigServiceUtil.getGdpAndError(site, QmqType.CREATE_ORDER, ConfigType.ERROR);
            }
          },
          PURCHASE(".purchase", "采购", 99, 99) {
              @Override
              public int getGdp(String site) {
                  return ConfigServiceUtil.getGdpAndError(site, QmqType.PURCHASE, ConfigType.GDP);
              }

              @Override
              public int getMaxError(String site) {
                  return ConfigServiceUtil.getGdpAndError(site, QmqType.PURCHASE, ConfigType.ERROR);
              }
          },
          TICKET(".ticket", "出票", 99, 99) {
              @Override
              public int getGdp(String site) {
                  return ConfigServiceUtil.getGdpAndError(site, QmqType.TICKET, ConfigType.GDP);
              }

              @Override
              public int getMaxError(String site) {
                  return ConfigServiceUtil.getGdpAndError(site, QmqType.TICKET, ConfigType.ERROR);
              }
          };

          // redis中存的类型后缀
          public String VALUE;

          // 发短信中的类型说明
          public String NAME;

          // config中没有配置或者配置出错，提供的default监控值
          public final int DEFAULT_GDP;

          // 同上,default允许的失败值
          public final int DEFAULT_ERROR;

          public abstract int getGdp(String site);

          public abstract int getMaxError(String site);

          QmqType(String value, String name, Integer defaultGdp, Integer defaultError) {
              this.VALUE = value;
              this.NAME = name;
              this.DEFAULT_GDP = defaultGdp;
              this.DEFAULT_ERROR = defaultError;
          }
    }


2.  用实例域代替序数(用的较多了,一般都直接不使用序数)


        public enum CabinTypeEnum {
          FIRST_CLASS_CABIN(1),
          BUSINESS_CLASS_CABIN(2),
          ECONOMY_CLASS_CABIN(0);

          private final int code;

          CabinTypeEnum(int code) {
            this.code = code;

          }
          public int getCode() {
            return code;
          }     
        }
3.  用EnumSet代替位域,之前看到scheduler的设计很牛逼，这才发现还有更好的方式。


        // 不好
        public class Text {
          public static final int STYLE_BOLD = 1 << 0;
          public static final int STYLE_BOLD = 1 << 1;
          public static final int STYLE_BOLD = 1 << 2;
          public static final int STYLE_BOLD = 1 << 3;

          public void applyStyles(int xx) { ... }
        }

        text.applyStyles(STYLE_BOLD | STYLE_ITALIC);

        // 好
        public class Text {
          public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

          public void applyStyles(Set< Style > styles) { .. }
        }     

        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC) );

4.  用EnumMap来代替序数索引

    >   a.使用EnumMap来关联enum的数据 (eg: 多舱等,每个舱等对应多个舱位)

        public enum CabinTypeEnum {
          FIRST_CLASS_CABIN(1),
          BUSINESS_CLASS_CABIN(2),
          ECONOMY_CLASS_CABIN(0);

          private final int code;

          CabinTypeEnum(int code) {
            this.code = code;

          }
          public int getCode() {
            return code;
          }     
        }   
        // 不好
        Map<int,List<Cabin>> map = Maps.newHashMap();
        for(CabinTypeEnum cabinTypeEnum : enumMap.keySet()){
          map.put(cabinTypeEnum.getCode(),Lists.<Cabin>newArrayList());
        }
        map.get(CabinTypeEnum.FIRST_CLASS_CABIN);

        // 好
        Map<CabinTypeEnum,List<Cabin>> enumMap = Maps.newEnumMap(CabinTypeEnum.class);
        for (CabinTypeEnum cabinTypeEnum : enumMap.keySet()) {
          enumMap.put(cabinTypeEnum, Lists.<Cabin>newArrayList());
        }
        enumMap.get(CabinTypeEnum.FIRST_CLASS_CABIN);

    >   b.使用嵌套的EnumMap来关联enum pairs, (eg :水,气,冰  升华,熔化,凝华,液化..)

5.  用接口模拟可伸缩的枚举

eg: 一个操作的enum,里边实现了加减乘除,但是你又想加入一些额外的操作,取余,幂等操作,又想有所区别表示基本操作和额外操作,使用接口就太棒了

      public interface Operation {
        double apply(double x, double y);
      }

      public enum BasicOperation implements Operation {
        PLUS("+") { public double apply(double x, double y) { return x + y; } },
        MINUS("-") { public double apply(double x, double y) { return x - y; } },
        TIMES("*") { public double apply(double x, double y) { return x * y; } },
        DIVIDE("/") { public double apply(double x, double y) { return x / y; } };

        private final String symbol;

        BasicOperation(String symbol) { this.symbol = symbol; }

        @Override
        public String toString() {
            return symbol;
        }
      }

      // Extend from BasicOperation

      public enum ExtendedOperation implements Operation {
        EXP("^") { public double apply(double x, double y) { return Math.pow(x, y); } },
        REMAINDER("%") { public double apply(double x, double y) { return x % y; } };

        private final String symbol;

        ExtendedOperation(String symbol) { this.symbol = symbol; }

        @Override
        public String toString() {
            return symbol;
        }
      }

      public static void main(String[] args ){
        double x = 1.1;
        double y = 2.2;

        test(ExtendedOperation.class, x, y);
      }

      // 这个还没见过哎    &
      private static <T extends Enum< T > & Operation > void test( Class< T > opSet, double x, double y) {
        for (Operation op : opSet.getEnumConstants())
            System.out.printf("%f %s %f = %f\n", x, op, y, op.apply(x, y) );
      }

### 参考

Effective java

[1]<http://allenlsy.com/NOTES-of-Effective-Java-5/>
