---
title: "Writing unit-testable code"
author: yiqi
permalink: /blogs/2021-01-30-java-unit-test
date: 2021-01-30
collection: blogs
tag:
- Programming
- Java
---

- [Is your static method readlly STATIC?](#is-your-static-method-readlly-static)
- [Interfaces can be helpful](#interfaces-can-be-helpful)

Recently I was doing some modification of our core service package. The code change should have been pretty straightforward if the existing code were written in a more testable way...

When wring a piece of code, we should always keep in mind that later we will write unit test for it. Additionally and more importantly, we'll test it's "dependers" and may need to mock this code functionality.

Unit test can be a very big topic. In this post let's visit some typical cases.

# Is your static method readlly STATIC?

Let's start with a toy example. Irrelevant details are hidden.  

```java
public class WeatherClient {

    public Weather getCurrentWeather() {
        /**
         * connect to server to get the current weather condition
        */
    }

}

public class DressingUtility {

    public static String getDressingSuggestion() {
        Weather weather = new WeatherClient().getCurrentWeather();
        switch (weather) {
            case Weather.SUNNY:
                return "T-shirts and shorts";
                break;
            case ... ...
            /**
             * more case statements...
            */
        }
    }

}

public class ChatBot {

    public String replyWeatherRequest() {
        /**
         * Reply with weather as well as some dressing suggestions
         * 1. get weather
         * 2. DressingUtility.getDressingSuggestion()
        */
    }

}
```

The code itself is quite trivial. Now let's think of unit test for the class ```ChatBot```. Apparently we need to mock ```DressingUtility.getDressingSuggestion()```; but how to mock a static method?

If you try to Google it you'll find Powermockito, a extension API which supports mock static functions. Problem solved! .... or is it?

If you spend some time on Powermockito, you probably will encounter annoying problems such as complex unit test code, incompatible version of Powermockito and Mockito, wired error and bugs due to reflection (Powermockito uses reflection to mock static methods).  

STOP HERE! If you are writing some code and realize that you need to mock static methods, this is a very strong sign that you probably just wrote bad code! Let me explain.  

What is utility method (or: what kind of methods can be static)? In short: stateless methods. In the above example, ```DressingUtility.getDressingSuggestion()``` is stateful becuase it's output depends on the result of ```getCurrentWeather()```. To mock the functionality of ```getDressingSuggestion()``` we need to somehow inject a mock of ```WeatherClient```...

A stateless method's output depends merely on the input. ***There is NO need to mock static methods at all as a good static function itself is a Mock!*** You can easily control its output just by passing in corresponding inputs.  

I myself always think of the following example when deciding whether to write static methods:  

```java
public class GoodStaticExample {

    public static calculateSum(int a, int b) {
        return a + b;
    }

}
```

# Interfaces can be helpful

```java
public class RuntimeInfoResolver {

    private RuntimeEnvInfo runtimeInfo;

    public RuntimeInfoResolver(RuntimeEnvInfo runtimeInfo) {
        this.runtimeInfo = runtimeInfo;
    }

    public String deriveSomethingFromRuntimeEnvInfo() {
        /**
         * code block that uses runtimeInfo
        */
    }
}
```

This example is a simplified version of a class I was writing in our service. ```RuntimeEnvInfo``` contains internal functionality to derive runtime information of our service. The required metadata doesn't exist in desktop enviromments so when running unit test ```RuntimeEnvInfo``` fails instantiation.  

The solution is to use interface.

```java 
public interface InfoResolver {

    public String deriveSomethingFromRuntimeEnvInfo();

}

public class RuntimeInfoResolver implements InfoResolver {
    /**
     * save as above
    */
}

public class BusinessLogicClass {

    private InfoResolver infoResolver;

    public BusinessLogicClass(InfoResolver infoResolver) {
        this.infoResolver = infoResolver;
    }

    /**
     * constructor, public methods, etc...
    */
}
```

When unit testing ```BusinessLogicClass```, we can declare a completely fake info resolver class instead of using ```RuntimeInfoResolver```. This way we avoid instantiating ```RuntimeEnvInfo```.

```java
public class BusinessLogicClassTest {

    private static class FakeRuntimeInfoResolver implements InfoResolver {

        public String deriveSomethingFromRuntimeEnvInfo() {
            /**
             * Mock logic; return anything needed
            */
        }

    }

    @Test
    public void TestSomeMethod() {
        BusinessLogicClass businessLogic = new BusinessLogicClass(new FakeRuntimeInfoResolver());
        /**
         * unit test logic
        */
    }

}
```
