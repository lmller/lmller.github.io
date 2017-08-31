---
layout: post
title:  "Improving the Readability of Mockito Tests with Kotlin"
date:   2017-07-03 15:00:00
description: Mockito and other mocking frameworks are powerful testing tools. Clean up the code that is necessary to setup mocking or stubbing with Kotlin extension functions.
comments: true
categories:
- kotlin, test
permalink: kotlin-mockito-readbility
---

Mockito(todo, link) is a very powerful and flexible tesing tool that can be used to easily stub(2) or mock(3) required dependencies in unit tests.  
One disadvantage of tools like this is that they come with a huge set of methods, classes and instructions and actually define a language of its own.  Reading code that depends on mocks can be hard. Let's improve this with Kotlin's extension functions.

Imagine I want to test an object that depends on the results of a server call (or any other data source). 

```kotlin
class Contract (val customerId: Long, val contractApi: ContractApi) {
    var monthlyFee: BigDecimal = BigDecimal.ZERO

    fun load() = contractApi.getContract(customerId)
        .subscribe(
            { contract -> monthlyFee = contract.yearlyAmount / BigDecimal(12) }
            { println("It's an error!") }
        )
}

... 

interface ContractApi {
    @GET fun getContract(@Query("customerId") customerId: Long) : Single<ContractDto>
}
...
data class ContractDto(val dataBaseId: Long, val yearlyAmount: BigDecimal)

```
Note that I'm using Retrofit and RxJava to define the Api for convenience.  

Now, I want to test that the monthly fee is calculated correctly:

```kotlin
...
@Test fun `Contract stores the monthly fee`(){
    val apiMock = mock(ContractApi::class.java)
    val contract = ContractDto(1337, 99.99)
    val result = Single.just(contract)

    given(apiMock.getContract(1337)).willReturn(result))

    val sut = Contract(1337, apiMock)

    assertThat(sut.monthlyFee, equalTo())
    
}