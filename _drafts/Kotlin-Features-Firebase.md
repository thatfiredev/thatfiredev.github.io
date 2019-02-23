---
layout: post
title:  "Kotlin Extension Functions e Coroutines no Firebase"
date:   2019-02-23 22:44:00 +0200
categories: firebase
---

> "Quanto mais simples e conciso for o código, mais rápido você entenderá o que está acontecendo." - 
> [Paulo Enoque](https://medium.com/@pauloenoque2014)

É por causa da simplicidade e concisão do
[Kotlin](https://medium.com/kotlin-para-android/introdu%C3%A7%C3%A3o-a-kotlin-51a4355f3bc8) que muitos desenvolvedores
 têm optado por esta linguagem de programação para realizarem o seu trabalho.
Devo recordar também que em Maio de 2017 a Google anunciou que Kotlin passava a ser uma linguagem oficial para
 desenvolvimento de aplicações Android. De lá para cá, a adoção desta linguagem e, consequentemente, o número de
  desenvolvedores Kotlin também aumentaram.
Se você ainda não é um destes desenvolvedores, mas gostaria de conhecer o Kotlin, recomendo que leia esta série de
 artigos do [Paulo Enoque](https://medium.com/@pauloenoque2014):

1. [Introdução a Kotlin](https://medium.com/kotlin-para-android/introdu%C3%A7%C3%A3o-a-kotlin-51a4355f3bc8)
2. [Sintaxe básica do Kotlin](https://medium.com/kotlin-para-android/sintaxe-b%C3%A1sica-do-kotlin-7548a441f5e5)
3. [Detalhes avançados sobre Kotlin](https://medium.com/kotlin-para-android/detalhes-avan%C3%A7ados-sobre-kotlin-c0a9b0059880)
4. [Kotlin Android Extensions](https://medium.com/kotlin-para-android/kotlin-android-extensions-8abdc33f21ef)

---

Só um ano e meio depois do Kotlin ser adoptado como linguagem oficial para o Android é que o Firebase incluiu esta
 linguagem na sua [documentação oficial](https://firebase.google.com/docs) :

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/Firebase?ref_src=twsrc%5Etfw">@Firebase</a> code samples for Android are starting to get <a href="https://twitter.com/kotlin?ref_src=twsrc%5Etfw">@kotlin</a> samples to go with Java! Thanks to @daggerdwivedi and <a href="https://twitter.com/_rpfernandes?ref_src=twsrc%5Etfw">@_rpfernandes</a> for the assists here. It&#39;s a tremendous effort overall, and it&#39;s great to have help from the community.<a href="https://t.co/PMlAViMf9F">https://t.co/PMlAViMf9F</a><a href="https://twitter.com/hashtag/AndroidDev?src=hash&amp;ref_src=twsrc%5Etfw">#AndroidDev</a> <a href="https://t.co/W386VlM3HC">pic.twitter.com/W386VlM3HC</a></p>&mdash; Doug Stevenson 🔥 (@CodingDoug) <a href="https://twitter.com/CodingDoug/status/1050473809994629120?ref_src=twsrc%5Etfw">October 11, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Mas como diz o ditado popular: "Mais vale tarde do que nunca". Esta alteração trouxe várias melhorias para a plataforma:
 o código Java do [Firebase Android SDK](https://github.com/firebase/firebase-android-sdk) foi alterado para melhor acomodar a interoperabilidade que o Kotlin oferece,
 "Extension Functions" foram criadas para tornar o uso do SDK mais conciso no Android, desenvolvedores criaram bibliotecas
 para melhorar a forma como o Firebase é utilizado na linguagem Kotlin, e muito mais.

Eu juntei-me à criação de bibliotecas para ajudar a utilizar o Firebase em Kotlin, desenvolvi a biblioteca
 [fireXtensions](https://github.com/rosariopfernandes/fireXtensions) que oferece um conjunto de **Extension Functions**
 para o Firebase.

# Kotlin Extension Functions

De forma resumida, Extension Functions é uma funcionalidade do Kotlin que permite que você adicione novos métodos à
 classes que não são suas, sem ter de herdar (o famoso extends do java) dessa classe. Você pode saber mais sobre
 Extension Functions no meu artigo [Apresentando o fireXtensions](https://medium.com/android-dev-moz/firektx-2fc1651d095e).

Depois de adicionar suporte para o Kotlin na Documentação Oficial, a equipa do Firebase decidiu adicionar também algumas
 Extension Functions no SDK Android do Firebase (pela mesma razão que eu criei o fireXtensions).

## Common KTX

O módulo firebase-common-ktx contém extension functions usadas para obter a instância do Firebase. Se você já trabalhou
 com o Firebase tanto em java, como em Kotlin, provavelmente está familiarizado com o método:
 `FirebaseApp.getInstance()`. Pois é, se você utilizar o módulo firebase-common-ktx, a chamada do método torna-se mais
  simples: `Firebase.app`.

Para utilizar este módulo, certifique-se que o seu build.gradle(project) tem a dependencia do Kotlin Plugin 1.3.20 ou
 superior:
 
```groovy
dependencies {
    classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.20"
}
```

E de seguida adicione a dependência no ficheiro build.gradle (app) do seu projeto:

```groovy
dependencies {
    // ... Outras dependencias ...
    implementation 'com.google.firebase:firebase-common-ktx:17.0.0-alpha01'
}
```

Veja abaixo algumas das funções disponíveis neste módulo:

```kotlin
// Sem o Common KTX:
val firebase = FirebaseApp.getInstance()

// Usando o Common KTX:
val firebase = Firebase.app



// Sem o Common KTX:
val firebase = FirebaseApp.getInstance(name)

// Usando o Common KTX:
val firebase = Firebase.app(name)



// Sem o Common KTX:
val firebase = FirebaseApp.initializeApp(context)

// Usando o Common KTX:
val firebase = Firebase.initialize(context)



// Sem o Common KTX:
val firebase = FirebaseApp.initializeApp(context, options)

// Usando o Common KTX:
val firebase = Firebase.initialize(context, options)



// Sem o Common KTX:
val firebase = FirebaseApp.initializeApp(context, options, name)

// Usando o Common KTX:
val firebase = Firebase.initialize(context, options, name)



// Sem o Common KTX:
val firebaseOptions = FirebaseApp.getInstance().options

// Usando o Common KTX:
val firebaseOptions = Firebase.options
```

Apesar de este ser o único módulo disponível atualmente, existem mais módulos que serão certamente adicionados ao SDK.
 Irei atualizar este artigo sempre que forem adicionados novos módulos.

# Kotlin Coroutines

Como você já deve saber, o [Firebase funciona de forma assíncrona](https://pt.stackoverflow.com/a/278696/39181).
 Isso leva-nos a utilizar listeners no nosso código para ler os dados da base de dados. E um erro bastante comum entre
 programadores iniciantes é tentar ler estes dados sem listeners, ou usá-los fora dos listeners, como mostra o exemplo 1:

**Exemplo 1:** Carregar a lista de todos utilizadores guardados no Firestore.
