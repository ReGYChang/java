# 相遇

## 淺談Framework
框架是一個相對抽象的概念，簡單來說是大家對於複雜的軟體系統提出一套`設計模式的思想`或是`規範`，可以大大提升系統的`開發效率`、`可維護性`、`擴展性`等等。

舉例來說，今天建設公司要蓋大樓，一定會參考業界的建築`規範`跟`設計模式`，不會因為要蓋一棟新的大樓而**重新規劃**所有的建築規範，因為那都是前人設計師用血淚總結出來的經驗與架構，我們要思考的是站在現有框架的角度上如何`優化`或`擴展`，而不是**重複的製造輪子**，這也是`Spring Framework`很重要的核心精神之一。

## Spring
Spring Framework 是一個開源的Java框架，號稱是世界上最受歡迎的Java framework，對Web application提供了大量擴展功能。理論上所有的Java application都可以導入Spring並應用其核心功能。其核心功能為提供`IoC Container`與`AOP`模組。

## 為甚麼需要Spring
前面提到了，框架是為了提高開發效率而引入一連串的規範跟思想，而Spring主要幫我們做到了以下幾件事:
- 基於`POJO輕量級`與`非侵入式`開發
- 通過`IoC`與`DI`、`抽象化`的設計思想來實現系統的`低耦合`
- 基於`AOP`和`約束`實作`宣告式程式設計`
- `集成框架`支持

## 名詞解釋
### POJO
全名是Plain Ordinary Java Object，就是簡單的Java物件。含意是指沒有從任何類別繼承、也沒有實作任何界面的Java物件。
### 侵入式概念
**侵入式**
- 改變了Java class的結構
- 對於`EJB`、`Struts2`等傳統框架來說通常要**實作特定的介面**或**繼承特定的類別**才可以實現功能的擴展

**非侵入式**
- 對於`Spring`來說則不需要對現有class結構做出改變即可增強JavaBean功能
  
### 低耦合
耦合是指系統間各種不同功能的`模組`之間關聯度量。耦合程度的高低取決於模組間的`控制關係`、`調用方式`來決定。

### 抽象化
抽象化可以總結成**從實體當中萃取出更高層的概念**。譬如以前國中生物學學到的生物分類法，將所有生物歸類為「界門綱目科屬種」。「界門綱目科屬種」為抽象化的展現，將具備部分相同特徵(**屬性或方法**)的生物(**物件**)層層抽象歸類。

在Java中抽象化主要可以通過繼承`abastract class`或實作`interface`去實現。兩者的差異大致是abstract class抽象了類別(**屬性與方法**)，而interface抽象了行為(**方法**)。

---
# 相知

## Spring組成與架構
Spring主要分為八大模組:
- Spring Core - 提供IoC Cotainer，負責管理物件的創建與依賴
- Spring Web - Spring對`web`模組的支持
  - 可以與`struts2`整合，讓`struts2`創建的`Action`交由Spring管理
  - 整合`Spring MVC`框架
- Spring DAO - Spring對資料封裝的支持
- Spring Aspects
- Spring Instrumentation
- Spring Messaging
- Spring AOP - `剖面導向程式設計`
- Spring Test

其中最核心的兩個模組分別是`Spring IoC`與`Spring AOP`，以下分別將兩個模組拉出來介紹說明。

## IoC

### 概念
在開始前我們先思考:
- 物件創建...
---

IoC全稱`Inverse of Control`，是Spring的核心思想之一。

主要思想是將物件之間的依賴跟創建的控制權從物件中抽離，轉交給`IoC Cotainer`來管理跟控制，資源不再由使用資源的雙方管理，而交由不使用資源的第三方管理。主要有以下優點:
- 資源集中管理，實現**可配置性**與**易維護性**
- 大幅降低資源雙方耦合度，達到**解耦**目的

而前面略提到的`DI(Dependency Injection)`則是實現IoC思想的一種方式。
- Dependency : bean物件的**創建依賴**於Container
- Injection : bean物件中的**屬性**由Container注入

在Java中，當A模組要使用到B模組功能時，通常是這麼做的:
```Java
public B class(){
  //code...
  A a = new A();
  //code...
}
```
這種耦合在軟工中稱為`Content Coupling`，指一個模組**直接呼叫**另一模組的內容。這種耦合的`耦合性最強`，`模組獨立性最弱`。這不是我們所樂見的。透過`Spring IoC`來管理並調用模組，就可以解決`Content Coupling`的問題。無論是**創建物件**、**處理物件間的依賴關係**、**物件創建的時間**還是**物件的數量**，我們都可以通過Spring提供的`IoC Container`來管理。
### [延伸閱讀 - Spring IoC](#)

<!-- ### IoC創建物件方式   Spring IoC內容
- how Spring Works
- non-argument constructor(Default)
- argument constructor
  - constructor argument index
  - constructor argument type matching(no recommended)
  - constructor argument name

> Note: 在配置文件加載時，IoC Container所管理的全部物件就已經實體化

### 獲取IoC Container物件
### Spring配置
### DI
### Bean Autowire
### Annotaion
### JavaConfig -->

## AOP
### 概念
首先我們先思考一個問題:

今天我們希望在我們的應用程式中加入**權限驗證**，舉例來說有些東西只有老闆可以看，一般員工不行。

如果依循`OOP物件導向`的思維，我們就會在業務邏輯程式碼前再加上驗證的程式碼去確認身分並做出判斷。

但如果基於`AOP剖面導向`的思維，切開業務邏輯與驗證邏輯，把驗證邏輯放到業務邏輯之外去完成，就是AOP的核心概念。他與OOP一樣都是`程式設計典範`。

一般來說我們的程式執行流程是由上而下的，譬如一個登入邏輯:
- User發送HTTP request
- Controller接收request並呼叫對應的Service
- Service呼叫DAO操作DB並return
- 層層return給User

但在這過程中會產生一些橫切性的問題，譬如進入Controller時我們需要**紀錄日誌**、DAO操作DB時希望進行**Transaction管理**、進入Service時我們希望做**身分驗證**等等等等...

AOP即是為了解決這種橫切性的問題而誕生。AOP不關心業務邏輯，只關心這些橫切性問題的**執行方法**、**執行時機**、**執行的地方**、**執行順序**等等。

### [延伸閱讀 - Spring AOP](#)

# 相惜
這篇文章主要簡單介紹Spring並針對其兩個核心設計思想的概念做一個介紹，更多的設計思想實作會再通過一系列的文章慢慢介紹。如果覺得有幫助到你，可以幫我點個讚小小支持。也歡迎在底下留言交流。
> 本文已被收錄在[Github](#)
