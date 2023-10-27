---
title: MVC,MVP,MVVM
auther: ckh
date: 2023-08-16 23:47:10 +0800
categories: [Programming, Architect]
tags: [Architect]    
---
  
# UI design pattern  
UI 를 디자인하는 방법론은 대표적으로 MVC, MVP, MVVM 3가지 이다.  
현재에는 각 패턴이 쓰이던 ``문제점을 보완``하여 사용된다. 이로 인해 각 패턴사이의 경계가 굉장히 모호해졌다. (특히 MVC와 MVP는 거의 비슷하다)  
이 글에서는 처음 모습의 패턴을 알아보고, 어떻게 보완되었는지 간단히 알아본다.  

    
# MVC
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/a919817b-5011-4f33-af10-b9d60f62e9c3)  
  
Controller가 Model, View에 의존한다.  
  
> 모든 유저 input/output이 controller를 통해 전달된다.  
> controller는 받은 input에 따라 가지고 있는 Model, View에 변경사항을 적용한다.
controller가 굉장히 비대해지며 여러 view에 의존하는 경향이 있다.
코드 재사용성이 떨어지며, 프로젝트가 커질수록 controller도 비례하여 커진다.
  
  
## code  
JS로 이루어진 아주 간단한 예시코드
  
```javascript
class TaskModel {
    constructor() {
        this.tasks = [];
    }

    addTask(task) {
        this.tasks.push(task);
    }

    getTasks() {
        return this.tasks;
    }
}
```
  
```javascript
//View
class TaskView {
    constructor() {
        this.taskList = document.getElementById('taskList');
    }

    showTasks(tasks) {
        this.taskList.innerHTML = '';
        tasks.forEach(task => {
            const li = document.createElement('li');
            li.textContent = task;
            this.taskList.appendChild(li);
        });
    }
}
```
  
```javascript
//controller code
class TaskController {
    constructor(model, view) {
        this.model = model;
        this.view = view;
        
        this.taskInput = document.getElementById('taskInput');
        this.addTaskButton = document.getElementById('addTaskButton');
        
        this.addTaskButton.addEventListener('click', () => {
            this.addTask(this.taskInput.value);
        });
    }

    addTask(task) {
        this.model.addTask(task);
        this.view.showTasks(this.model.getTasks());
    }
}
```
  
```javascript
//init
const model = new TaskModel();
const view = new TaskView();
const controller = new TaskController(model, view);
```
  
## 설명
* 테스트 및 변경에 좋지 않다.
 * controller 테스트시, user input을 목업하기 힘들다.
  * user가 보는 view가 controller와 강하게 결합 되어있다. 이는 user input 자체를 simulate해야 하는 상황이 생기기 쉽다.
  * controller를 테스트하기 위해 userinput을 controller가 받아 model과 view를 조작하므로, 코드가 복잡할수록 나눠서 테스트하기 힘들다.
 * controller 변경시, viewer를 같이 변경할 가능성이 굉장히 높다. 상황에 따라 model변경도 같이 필요 할 수 있다.
* 코드 재사용성이 떨어진다.
 * user input을 controller가 받으므로, 거의 view와 controller가 분리되지 않는다.

위와 같은 문제점을 보완하기 위해 MVP 방법론이 만들어 졌다.  
(다시한번 말하자면 위의 문제는 최근에 쓰는 MVC모델에서도 개선된 상태로 쓰이는 경우가 많다.)  
  
# MVP
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/22d6b073-8f82-4401-909f-501b011da453)
* MVP design은 controller가 방대해지는 것을 막고, view와 controller의 애매한 결합을 좀더 명확한 역할 분리를 하였다.  
View에서 User input/output을 담당하나, 직접적인 처리는 Presenter에 위임(delegate)한다.  
Presenter는 Controller와 거의 같은 역할이다. View와 결합도를 낮추었는데, 이는 View, Presenter 각각 서로의 instance를 이용하여 api 호출 방식으로 통신한다.  
presenter는 View의 정확한 사용방식을 구현하지 않고, api를 정의하는 방식으로 구현된다.  
View는 정의된 api를 호출하는 식으로 presenter를 사용하므로써 서로 인터페이스(api)에 의존되는 형식으로 변경하여 결합도를 낮춘다.  
* 기본적으로는 View와 presenter가 1:1로 의존한다. presenter가 비대해지기보다, view하나당 presenter하나식으로 결합되므로 크기도 작아진다.  
## code  
```javascript
//model
class TaskModel {
    constructor() {
        this.tasks = [];
    }

    addTask(task) {
        this.tasks.push(task);
    }

    getTasks() {
        return this.tasks;
    }
}
```
```javascript
//view
class TaskView {
    constructor(presenter) {
        this.presenter = presenter;
        this.addTaskButton = document.getElementById('addTaskButton');
        this.taskInput = document.getElementById('taskInput');
        this.taskList = document.getElementById('taskList');
        
        this.addTaskButton.addEventListener('click', () => {
            this.presenter.handleAddTask(this.taskInput.value);
        });
    }

    showTasks(tasks) {
        this.taskList.innerHTML = '';
        tasks.forEach(task => {
            const li = document.createElement('li');
            li.textContent = task;
            this.taskList.appendChild(li);
        });
    }
}
```
```javascript
//Presenter
class TaskPresenter {
    constructor(model, view) {
        this.model = model;
        this.view = view;
    }

    handleAddTask(task) {
        this.model.addTask(task);
        this.view.showTasks(this.model.getTasks());
    }
}
```  
view의 역할 및 관계
* Presenter에 의존하여 api의 형태로 작업을 넘긴다.
* 유저 인풋을 view에서 받아서 처리를 presenter에 위임한다
* 아웃풋을 render를 하는 역할을 한다.
  
Presenter의 역할  
* View, Model에 의존하여 작업에 따라 model, view를 적절히 사용하여 view에 랜더링한다. 랜더링은 view의 api를 이용한다.  
* view에게 위임받은 일을 처리한다. api형태로 받아 처리하므로 테스팅이 이전보다 쉬워진다.

## 정리
1. api형태로 서로를 호출하므로써, 변경/추가 기능이 좀더 명확하다.
2. view와 presenter가 1:1로 매칭된다. 또한 각각 api호출처럼 되어있어 재사용성도 좀더 쉬워진다.  
  
# MVVM
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/9b29c5c3-16dd-425a-bcd4-788a26530572)
ViewModel이 더이상 View에 의존하지 않는다.  
* pure한 MVVM패턴을 만들기 위해서는 Observer pattern 혹은 event등을 사용하여 구현된다.
ViewModel은 여러 View에서 사용이 가능하다.
* 디자인적으로 ViewModel이 View에 직접 의존하지 않으므로, View에서 closure함수처럼 사용이 가능하다.

## code
카운터를 만드는 코드이다.  
```javascript
// Model
const model = {
    count: 0
};

// ViewModel
const viewModel = {
    increment: function() {
        model.count += 1;
        this.notify('countChanged', model.count);
    },
    decrement: function() {
        model.count -= 1;
        this.notify('countChanged', model.count);
    },
    listeners: {},
    on: function(event, callback) {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }
        this.listeners[event].push(callback);
    },
    notify: function(event, data) {
        if (this.listeners[event]) {
            this.listeners[event].forEach(callback => callback(data));
        }
    }
};

// View
document.querySelector('[data-bind="counter"]').textContent = model.count;

viewModel.on('countChanged', (newCount) => {
    document.querySelector('[data-bind="counter"]').textContent = newCount;
});
```  
viewModel이 Model과 event(+callback)를 통해 구현됬다.  
View는 이 viewModel에 callback을 사용하여 구현된다.  


