---
layout: post
---

## Dependency Injection
what is this dependency injection that everyone is talking about.  
To understand this first we must know what is dependency.  

In simple language when ever we do something like
```
class DoSomethingWithAnimal {
  Dog dog;
  public DoSomethingWithAnimal() {  
    dog = new Dog();
  }

  public void animalVoice() {
    dog.makeSound();
  }
}
```
we create a dependency of **Dog** on the class **DoSomethingWithAnimal**. In other words **DoSomethingWithAnimal** is responsible for the creation of **Dog**. This is what we call dependency.  
You might ask what is the problem with this, why so much fuss about this?  
Let me tell you the problem
In future if we want to replace the Dog with cat or say your favourite animal then we will have to change the code and changing code is considered sin in programming as it leads to bugs. Remember >> Code should be open for extension but closed for modification.    

To solve this problem we use interfaces.
```
interface Animal{

}

class DoSomethingWithAnimal {
  Animal animal;
  public DoSomethingWithAnimal() {  
    animal = new Dog();
  }

  public void animalVoice() {
    animal.makeSound();
  }
}

```

In this when ever we want to change the **Dog** with **Cat** we will  have to change the code but just the line `animal = new Dog()` to whatever animal you want.  
But still a change is a change and a sin is a sin. What if we could remove `new Dog()` some how.  
This can be done by passing the instance of `Dog` through constructor.

```
class DoSomethingWithAnimal {
  Animal animal;
  public DoSomethingWithAnimal(Animal myAnimal) {  
    animal = myAnimal;
  }

  public void animalVoice() {
    animal.makeSound();
  }
}
```

Congratulations the problem is fixed. This is what is dependency injection. We inject the dependency and not create it, in this case we did it through constructor.
This type of code is also very useful when we want to write *unit tests* as we can just mock the animal and pass it in the constructor.
This is a very basic example explaining dependency injection and this becomes more and more complicated as we move towards a real world scenario. But there are libraries which helps us like Dagger (Check my blog about this [here](https://shubhamdhabhai.github.io/2018/01/24/Understanding-Dagger.html)), RoboGuice etc. They are all based on this simple concept and if you understand this you can understand them also.  

May the force be with you!!
