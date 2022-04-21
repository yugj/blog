# 字节码替换

## demo
```
// 替换方法
protected void doTransform() {
        ByteBuddyAgent.install();
        new ByteBuddy().redefine(ReflectUtils.class)
            .method(ElementMatchers.named("isPrimitive"))
            .intercept(MethodDelegation.to(MyReflectUtils.class))
            .make()
            .load(Thread.currentThread().getContextClassLoader(), ClassReloadingStrategy.fromInstalledAgent());
    }

public static class MyReflectUtils {
    public static boolean isPrimitive(Class<?> cls) {
        // xxx do sth
    }

}

// 替换字段值
ByteBuddyAgent.install();
        new ByteBuddy().redefine(AbstractDataBufferDecoder.class)
                .field(ElementMatchers.named("maxInMemorySize")).value(256 * 1024 * 1024)
                .make()
                .load(Thread.currentThread().getContextClassLoader(), ClassReloadingStrategy.fromInstalledAgent());
```