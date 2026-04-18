# Gradle

gradle打包azkaban报错：

```bash
> Task :azkaban-common:test 

azkaban.trigger.BasicTimeCheckerTest > testPDTtoPSTdst1 FAILED
    java.lang.AssertionError
        at org.junit.Assert.fail(Assert.java:86)
        at org.junit.Assert.assertTrue(Assert.java:41)
        at org.junit.Assert.assertTrue(Assert.java:52)
        at azkaban.trigger.BasicTimeCheckerTest.testPDTtoPSTdst1(BasicTimeCheckerTest.java:191)

azkaban.trigger.BasicTimeCheckerTest > testPDTtoPSTdst2 FAILED
    java.lang.AssertionError
        at org.junit.Assert.fail(Assert.java:86)
        at org.junit.Assert.assertTrue(Assert.java:41)
        at org.junit.Assert.assertTrue(Assert.java:52)
        at azkaban.trigger.BasicTimeCheckerTest.testPDTtoPSTdst2(BasicTimeCheckerTest.java:234)

azkaban.trigger.BasicTimeCheckerTest > testPDTtoPSTdst3 FAILED
    java.lang.AssertionError
        at org.junit.Assert.fail(Assert.java:86)
        at org.junit.Assert.assertTrue(Assert.java:41)
        at org.junit.Assert.assertTrue(Assert.java:52)
        at azkaban.trigger.BasicTimeCheckerTest.testPDTtoPSTdst3(BasicTimeCheckerTest.java:272)
```

