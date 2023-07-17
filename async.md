---
title: " javascript 非同期関数入門"
emoji: "🔗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript"]
published: false
---

## はじめに
**目的：**  
setTimeout, async-await, promiseを使いこなす  
asyncの意味を理解する  
awaitを使うべきタイミング、使うべきでないタイミングを知る
promiseの解決の概念を知る

**対象者：**
* 非同期関数が全然わからない人
* 非同期関数が何となくわかってきたけど、やってみるとうまくいかない人
* 非同期関数を使ってるのに同期的になってしまう人
* .thenのネスト地獄になっている人
* 非同期関数が全然わからない人に非同期関数を教えたい人

**目標：**  
まずはやりたいことをはっきりさせよう。  
今回は、以下の要件を満たすプログラムを実際に作ってみる。

1. プログラム開始の合図
1. ランダムで0~1秒後にHelloWorldする
1. HelloWorldとは関係なく、プログラム開始から1秒後にメッセージが送られる
2. HelloWorldの1秒後にメッセージが送られる

イメージはこんな感じ
```
=== プログラム開始 ===  // t = 0
Hello World!           // t = 0.3(ランダム)
開始1秒後               // t = 1.0
ハロワ1秒後             // t = 1.3
```
イメージはつかめただろうか。

## async-await, promise.thenは非同期関数を同期的に使う方法である
async-await, promise.thenは非同期関数を同期的に使う方法だ。
例えば、
```js:example1-discord.js
function replyOnReacted(message){
  message.createReactionCollector({filter(reaction){ return reaction === "⭕" }}).on("collect", reaction => {
  reaction.message.channel.send("good reaction!")
})
}
const message = await channel.send("hello!")
message.createReactionCollector({filter(reaction){ return reaction === "⭕" }}).on("collect", reaction => {
  
})
```


## 非同期を使わない方法
技術的には可能（=ゴリゴリ計算すればやってやれんことは無い）。
```js:非同期を使わない
const randomTime = Math.random()*1000

console.log("=== プログラム開始 ===")
sleep(randomTime)
console.log("Hello World!")
sleep(1000 - randomTime)
console.log("開始一秒後")
sleep(randomTime + 1000)
console.log("ハロワ一秒後")

function sleep(time){
  const start = Date.now() // new Date()と同じ意味。whileの前に定義して、スタート地点の時刻を固定しておく。
  while(Date.now() - start < time){} // timeミリ秒間空関数をグルグル。timeミリ秒後にループを抜けて、sleep()は処理を終了する。
}
```
console.log()がすぐに終わっているからよいものの、API叩く処理など不定の時間がかかる処理を挟む場合は使えない。
また、「ランダム」の具体的な値を取得できない場合も使えない。

## 強引な方法(setTimeoutのみ)
javascriptで非同期タイマーと言えば、setTimeoutが使える。  
setTimeoutは、何秒後に○○してくださいねと予約する関数だ。  
setTimeoutのみを使って実装してみる。
```js:setTimeoutのみ
console.log("=== プログラム開始 ===")
setTimeout(()=>{
  console.log("開始一秒後")
},1000)
setTimeout(()=>{
  console.log("Hello World!")
  setTimeout(()=>{
    console.log("ハロワ一秒後")
  },1000);
}, Math.random()*1000)
```
ちょっとネスト地獄が垣間見える。
なんなら「[非同期を使わない](#非同期を使わない方法)」方が見やすいまである。

## promise.then
今回の要件なら一番スマートなのが、thenを使う方法だ。
```
function sleepAsync(time){ return new Promise(resolve => setTimeout(resolve, time) }
// sleepAsync(1000).then(fn) は
// setTimeout(fn,1000) とおなじ

console.log()
sleepAsync()

console.log("=== プログラム開始 ===")
sleep(randomTime)
console.log("Hello World!")
sleep(1000 - randomTime)
console.log("開始一秒後")
sleep(randomTime + 1000)
console.log("ハロワ一秒後")
```



