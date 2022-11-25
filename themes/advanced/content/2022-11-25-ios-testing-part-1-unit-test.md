---
title: iOS Testing Part 1 - Unit Test
description: iOS Unit Testing Introduction
datePublished: 2022-11-25T12:22:28.436Z
dateModified: 2022-11-25T12:22:28.448Z
cover: https://images.unsplash.com/photo-1587620931276-d97f425f62b9?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2531&q=80
coverAlt: cover
slug: ios-testing-part-1-unit-test
category: iOS
tags:
  - testing
  - ios
---
U﻿nit testing is a great start to build the testing strategy for your App. Many people believe that writting Unit test will slow them down, but on the contrary they can significantly boost your productivity. We will not get into the technical details of adding Unit tests to your project, instead we will focus on the foundamentals.

## Why writing Unit tests 🤔? 

A﻿s a developer the least thing we want is unexpected behaviors when you push new code. It can lead to serious issues like app jcrash and bad reviews for your app. Sometimes you can introduce bugs that take you hours to resolve. By adding Unit tests you can have more confidence when making changes without breaking anything. They can also help you understand the current stability of your app.

Writing Unit tests can also help better structure your code. It makes you become the first customer of your own code, testing your components ensure their reliability and verifies the contract they promise. Spaghetti code will make it difficult to test, and thus force you to rethink about the architecture. It encurages people to write modular code which will also make it much easier to adding new features and making changes to the existing code.

They can also serve as the most updated documentation of your codebase. When working on big projects, it's hard to keep the documentation up to date, people are pushing new code on daily basis. Even worse when people switching teams or leave the company and context are lost. Unit tests can represent the current behavior of your codebase, give you an overview of how things should be working. 

> * Well tested code is easier to refactor or to add new functionality
> * They help you understand the current stability of your app
> * Unit test are trustworthy documentation of your codebase
> * If forces you to be the first consumer of your API and move faster

## What is Unit Testing 🔨?

W﻿hen building the app, compiler protects our codebase from syntax and build errors. Similarly well writting unit tests protect our codebase from logic errors. Here's an example of a simple Unit test. 

```swift
func test() {
  // given
  let price = sut.currentPrice + 5

  // when
  sut.setPrice(price)

  // then
  XCTAssertEqual(sut.price, 100, "Price doesn't match")
}
```

## **Unit Test Basic Concepts**

T﻿o fully understand how to write Unit test, we first need to learn a few concepts

**1. Target Class**

The class which the particular unit test is testing. Ideally, unit test files should focus on testing a single target class.

**2. Dependency**

An object which your target class interacts with. There are 3 main types of dependencies:

* Constructor Dependency - An object that is passed into a target class via a constructor
* Property Dependency - An object that is set on your target class through a setter (a property).
* Method Dependency - An object that is passed directly into the method you are invoking.

**3. Mocking**

A mock is a fake version of an object or interface which has the same API as the real version. In unit tests, every object that your target class interacts with should be mocked, i.e. all dependencies.

**4. Assertion**

Verify the correct behavior of a test. Similar to build errors from a compiler, logic errors from unit tests will block your commit

## **Phases of a Unit Test**

A single unit test run typically consists of 5 phases

1. Mock the target's constructor dependencies
2. Initialize target (with mocked dependencies)
3. Mock state of dependencies.
4. Invoke target API
5. Verify logic and state

## Case Study

L﻿et's take a look at an example. Here's the implementation of UserSettings

```objc
@implementation UserSettings

- (id)initWithJSONParser:(JSONParser *)parser
            userDefaults:(NSUserDefaults *)defaults {
  if (self = [super init]) {
    _parser = parser;
    _defaults = defaults;
  }
}

- (void)updateUserSettingsFromJSON:(NSString *)json {
  NSDictionary *newSettings = [_parser parseJSONDictionary:json];
  NSMutableDictionary *settings = [[_defaults objectForKey:kUserDataKey] mutableCopy];

  if (settings) {
    [settings addEntriesFromDictionary:newSettings];
  } else {
    settings = newSettings;
  }

  [_defaults setObject:settings forKey:kUserDataKey];
}

@end
```

A﻿nd let's see how should we add Unit test for this class

```objc
- (void)testUpdateUserSettingsFromJSON {
  NSString *json = @"{\"name\": \"Bill Gates\"}";
  NSDictionary *jsonDict = [NSDictionary dictionaryWithObject:
    @"Bill Gates" forKey:@"name"];

  // Phase 1: Mock target's dependencies
  id mockJSONParser = [OCMockObject mockForClass:[JSONParser class]];
  [[[mockJSONParser expect] andReturn:jsonDict]
    parseJSONDictionary:json];
    
  NSUserDefaults *userDefaults = [[NSUserDefaults alloc] init];
  
  // Phase 2: Init the target with mocked dependencies
  UserSettings *settings = [[UserSettings alloc]
    initWithJSONParser:mockJSONParser
          userDefaults:userDefaults];
  
  // Phase 3: Mock state of dependencies
  // In this case there's no state to mock 
  // as we already pass down all dependencies
  
  // Phase 4: Invoke the api
  [settings updateUserSettingsFromJSON:json];

  // Phase 5: Verify logic and state
  NSDictionary *result = [userDefaults objectForKey:kUserDataKey];
  STAssertEqualObjects(jsonDict, result, 
    @"User defaults should contain updated name");
}
```