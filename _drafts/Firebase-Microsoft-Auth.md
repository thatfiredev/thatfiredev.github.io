---
layout: post
title: "Firebase Authentication usando um Generic Identity Provider (IDP)"
date:   2019-03-07 02:43:00 +0200
categories: firebase
---

Desenvolver (do zero) um sistema de autenticação de utilizadores pode ser um bocado complexo, e quando queremos oferecer vários métodos de autenticação (ex. Email, Telefone, Google, Facebook, Twitter, etc) esta tarefa torna-se ainda mais complexa. Como resultado, acabamos nos focando mais no sistema de autenticação do que na aplicação que desenvolvíamos.

Para livrar-nos desta difícil tarefa, o Firebase contém o pacote [Authentication](https://firebase.google.com/docs/auth/), que é um serviço bastante simples para fazer a autenticação de utilizadores.

Desde que foi lançado, o Firebase Auth permite que utilizadores autentiquem-se na sua aplicação através de Email, Número de Telefone ou alguns dos *identity providers* mais populares como: [Google](https://medium.com/@mungoidario/firebase-auth-com-google-signin-ff89643322ab), [Facebook](https://medium.com/android-dev-moz/firebase-auth-facebook-login-6537e78456c6), Twitter e GitHub.

Mas hoje, o Firebase lançou uma nova funcionalidade que permite que você autentique-se com qualquer outro Identity Provider (IDP), como por exemplo Microsoft ou Yahoo. Decidi então experimentar esta funcionalidade e descrever a minha experiência neste artigo.

# Como Começar

Vou assumir que você já criou um projeto na [Consola do Firebase](https://console.firebase.google.com) e já [adicionou o Firebase à sua aplicação Android](https://firebase.google.com/docs/android/setup?hl=pt-pt).

## No Browser

Para iniciar, temos de criar uma aplicação no nosso Identity Provider (IDP) para obtermos o id (app id) e o segredo (app secret) da aplicação. Para esta experiência, decidi utilizar o IDP da Microsoft. Encontre abaixo os passos por mim seguidos (encontrados na documentação do [Firebase](https://firebase.google.com/docs/auth/android/idp-auth?hl=pt-pt) ):
1. Ir à [Microsoft Developer Console](https://apps.dev.microsoft.com) e criar uma aplicação (podia também usar uma aplicação existente). Isto gera um código denominado **Application Id**. Copie esse código.
2. Na secção **Application Secrets**, clique em **Generate New Password** e copie o resultado.
3. Na [Firebase Console](https://console.firebase.google.com), selecione o seu projeto e entre em **Authentication - Sign-in Providers**. Acione o método "Microsoft".
4. De seguida só tem de colar o Application Id e o Application Secret (password) que obteve da Microsoft Dev Console. A Firebase Console mostra então uma URL de redirecionamento, que você deve copiar.
5. Volte para a Microsoft Console e na secção **Platforms** clique em **Add Platform - Web** e cole a URL de redirecionamento (redirect URL) que você copiou da consola do Firebase.

## No código da aplicação

Agora que tem todo o backend preparado para autenticar utilizadores que utilizam contas Microsoft, tem de implementar o Firebase Auth no código da aplicação. Se você já utilizou outros métodos de autenticação com Firebase notará que o processo é praticamente o mesmo. Lembrando que estou assumindo que você já [adicionou o Firebase à sua aplicação Android](https://firebase.google.com/docs/android/setup?hl=pt-pt).

1. Adicione as dependências no ficheiro `build.gradle (app)` :
```groovy
    implementation 'com.google.firebase:firebase-core:16.4.0'
    implementation 'com.google.firebase:firebase-auth:16.4.0'
```
2. Crie uma variável para a instância do FirebaseAuth. Irei declarar como `lateinit` porque esta variável será inicializada no `onCreate()`:
```kotlin
class MainActivity : AppCompatActivity {
    private lateinit var auth: FirebaseAuth
    
    public override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Inicializar o Firebase Auth
        auth = FirebaseAuth.getInstance()
    }
}
```

3. No método `onStart()` da Activity, você pode verificar se já existe um utilizador logado. Caso exista, atualize a UI de forma apropriada:
```kotlin
    public override fun onStart() {
        super.onStart()
        // Verifica se já existe um utilizador
        val currentUser = auth.currentUser
        if (currentUser == null) {
            // Não existe nenhum utilizador
        } else {
            // Já existe um utilizador logado
        }
    }
```

4. Finalmente, crie um método para fazer o login. No meu caso chamei ele de `logIn()`:
```kotlin
   fun signIn() {
        val scopes = ArrayList<String>()
        
        val provider = OAuthProvider.newBuilder("hotmail.com", auth)
                                    .setScopes(scopes)
                                    .build() 

        auth.startActivityForSignInWithProvider(this, provider)
                .addOnSuccessListener { authResult ->
                    // Login efetuado com sucesso
                }
                .addOnFailureListener { e ->
                    // Ocorreu um erro durante o Login.
                }
    }
```

E pronto. Chegando a este ponto, você já deve ser capaz de autenticar utilizadores utilizando Microsoft Sign-in.
