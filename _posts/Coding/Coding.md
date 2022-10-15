# 代码规范
## if-else判断
### 冗余条件判断返回Boolean
```java
public class ReturnBooleanUglyCodeTest {
    @Test
    public void returnBoolean(){
        boolean good = goodReturnBoolean();
        boolean ugly = uglyReturnBoolean();
    }

    private boolean goodReturnBoolean(){
        List<Object> list = Lists.newArrayList("good");
        return list.iterator().hasNext();
    }

    private boolean uglyReturnBoolean(){
        List<Object> list = Lists.newArrayList("ugly");
        // 通过if判断返回Boolean型返回值时，此时应直接返回if判断的条件
        if(list.iterator().hasNext()) {
            // 而不应这样重复if判断后返回Boolean
            return true;
        } else {
            return false;
        }
    }
}
```

