---
layout: post
title: "Java to Scala: Test Object Builders"
date: 2013-10-20 14:57
comments: true
categories: java scala java-2-scala testing
published: false
---

=== Test Domain Object Generation in scala

builders

overview

Once you become test-infected a need quickly arises to facilitate the creation of test objects. As I made the transition from java to scala a very simple idiom emerged w/r/t test object generation that I would like to share. 

A popular approach in java is to create a builder which exposes a fluent interface. A fluent interface attempts to structure the api so client usage flows like natural language. From the client programmers perspective, it results in a humane api.  This is achieved by returning the builder itself from  each customization call. This allows you to chain the calls together in one statement which can sometimes form a complete sentence.

Let's examine an example of a credit card builder.


{% codeblock Simple domain object lang:java %}
public class CreditCard {
 
 private String nameOnCard;
 private Date expirationDate;
 private String cardNumber;
 private String cvv;

 public CreditCard(String name, String card, String cvv, Date exp) {
  this.nameOnCard= name;
  this.cardNumber = card;
  this.cvv = cvv;
  this.expirationDate = exp;
 }

}
{% endcodeblock %}


{% codeblock Typical builder lang:java %}
public class CreditCardBuilder {
 
 private String nameOnCard="Ed Rush";
 private Date expirationDate=new Date();
 private String cardNumber="0000000000000000";
 private String cvv = "112";

 public CreditCardBuilder withNameOnCard(String nameOnCard){
  this.nameOnCard=nameOnCard;
  return this;
 }
...todo
 public CreditCard build() {
  return new CreditCard(nameOnCard, cardNumber, cvv, expirationDate);
 }
}
{% endcodeblock %}

{% codeblock Fluent client example lang:java %}
CreditCard customTestCreditCard = new CreditCardBuilder()
  .withNameOnCard("Paul Rose")
  .withCardNum("4000400040004000")
  .withCvv("211") 
  .build();
{% endcodeblock %}

To scala

When I first started programming in scala, I pondered how one might implement the same style builder. In scala, due to a couple language features, the need for this pattern is almost eliminated. Named arguments allow you to specify a default value on functions. Objects with an apply method act as factories. The two of these together are a powerful combination. I would say this demotes the pattern to more of an idiom of the language.

{% codeblock Domain object in scala lang:scala %}
case class CreditCard(name: String, num: String, cvv: String, expiration: Date)
{% endcodeblock %}

{% codeblock Builder in scala lang:scala %}
object TestCreditCard{
 def apply(
  name: String="Ed Rush",
  num: String="0000000000000000",
  cvv: String="114",
  expiration: Date=new Date()
 ) = CreditCard(name, num, cvv, expiration)
}
{% endcodeblock %}

{% codeblock Builder client in scala lang:scala %}
val testCreditCardWithDefaults = TestCreditCard()
val testCreditCardWithCustomName = TestCreditCard(name="Paul Rose")
{% endcodeblock  %}

So, what does this buy us?
simple idiom, code reduction, terse, immutable
disadvantages: args derived from another, higher level customization functions, less fluent?

next: dynamic builders, generators, property testing & proofs
