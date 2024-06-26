Background processing
-  lifecycleScope
-  **ViewModelScope**
-  **LivedataScope**
-  **AsyncTask**
-  **Handlers e loopers**

<img src=".assets/166.jpg">

## Scope

- Sub-rotinas tem diversos tipos de escopo:
  -**GlobalScope**
    - Atrelado ao ciclo de vida da aplicação
  - **runBlocking**
    - Bloqueia a thread atual (Não recomendado!!!)
  - **lifeCycleScope**
    - Atrelado ao ciclo de vida da sub-rotina
  - **viewModelScope**
    - Atrelado ao ciclo de vida do ViewModel
  - **lifeDataScope**
    - Atrelado a visibilidade da UI


<img src=".assets/167.jpg">

<img src=".assets/168.jpg">

## ViewModel

<img src=".assets/169.jpg">

-  É um **detentor de estado** da tela ou da **lógica de negócios**

-  **Expõe** o **estado à interface** e **encapsula a lógica** de negócios correspondente

-  **Armazena** o estado em **cache** e **mantém** essas **informações** mesmo após **mudanças de configuração**

  - A **interface não precisa buscar dados novamente** ao navegar entre diferentes atividades ou implementar mudanças de configuração como ao girar a tela


## ViewModel - Atividade

- Criar uma classe kotlin que herda de ViewModel() com os seguintes códigos.

```kotlin
class FourthActivity_ViewModel: ViewModel(){
  private var counter = 0
  fun addOne(){
    counter++
}
fun getCounter(): String{
    return counter.toString()
}
```

- Instanciar o objeto com escope global e atrelar o texto do textView ao retorno da função get do contador do viewModel

```kotlin
//view model related
fourthAct_view_viewModel = ViewModelProvidaer(this).get(FourthActivity_ViewModel::class.java)
fourthAct_counter.text = fourthAct_viewModle.getCounter()
```

- No clique do botão chamar o a funçaõ addOne do ViewModel e atualizar o texto

```kotlin
R.id.fourth_act_add ->{
  //counter++
  //fourthAct_counter.text = counter.toString()
  fourthAct_viewModel.addOne()
  fourthAct_counter.text = fourthAct_viewModel.getCounter()
  }
}
```

- E agora?

  
<img src=".assets/170.jpg">

# LiveData

- Segue o padrão de projeto Observer

- Utilizado para atualizar uma UI
  - Atualiza apenas os observadores de componente do app que estão em um estado ativo no ciclo de vida
 
- Normalmente utilizado em conjunto com ViewModel

## LiveData - Atividade


<img src=".assets/171.jpg">

- Adicionar o seguinte código no viewModel

```kotlin

private var counter2 = MutableLiveData<Int>()
  fun getCounter2(): LiveData<Int>{
    return counter2
}
```

- Modificar o método addOne() no ViewModel

```kotlin
  fun addOne(){
    counter++
    counter2.value = counter
}
```

- Adicionar o seguinte método no onCreate() da atividade apóx a criação da instância do segundo textview

```kotlin
 //live data related
fourthAct_viewModel.getCounter2().observe(this,Observer{
  fourthAct_counter2.setText(fourthAct_viewModel.getCounter2().value.toString())
})
```

- Qual foi o resultado?


<img src=".assets/172.jpg">

# viewModelScope

```kotlin

fun addOne(){
  counter++
  viewModelScope.lauch{
    delay(1000)
    counter2.value = counter //atualiza synchronously
    counter2.postValue(counter) //atualiza asynchronously
}
}
```

## Scope

- Sub-rotinas tem diversos tipos de escopo:
  - **GlobalScope**
    - Atrelado ao ciclo de vida da aplicação
  - **runBlocking**
    - Bloqueia a thread atual (Não recomendado!!!)
  - **lifeCycleScope**
    - Atrelado ao ciclo de vida da sub-rotina
  - **viewModelScope**
    - Atrelado ao ciclo de vida do View Model
  - **liveDataScope**
    - Atrelado a visibilidade da UI
   
-->> TODAS RODAM NA MAIN THREAD !!! :000 <<--

## Dispatchers

- Determina qual thread a sub-rotina vai usar para a sua execução

  - **Dispatchers.Main**
 
  - **Dispatchers.IO**
 
  - **Dispatchers.Default**
 
  - **Dispatchers.Unconfined**
 
  - **Dispatchers.Main.immediate**

 ### Dispatchers.Main

 - Executa coroutines dentro da MainThread, ou seja, essa operação vai concorrer com suas operações de UI, como: Listeners, UI draws e tudo que envolve UI

 - **Aonde usar:**
   - Animações
   - sets de UI
   - buscas simples que retornam dados para UI
  
 ### Dispatchers.IO

  - Executa coroutines dentro de um conjunto de Threads separadas para IO

  - **Aonde usar:**
    - Chamdas de API
    - Chamadas de Banco de dados
   
  ### Dispatchers.Default

  - Executa coroutines dentro de um conjunto de Threads, assim como o IO, porém o foco dele é em operações pesadas do próprio app

  - **Aonde usar**
    - Operações em listas (sort, filter, map)
    - Transformações de imagem (resize, png to bitmap)
    
### Dispatchers.Unconfined

  - Executando com o Dispatcher que o chamar, e podendo ser finalizar em outro Dispatcher

    - **Aonde usar:**
      - Seu uso não é recomendado em código real !!!
     
### Dispatchers.Main.immediate

  - Immediate faz com que sua coroutine seja executada na hora que você a chamar, "passando" ela na frente de outra operações

  - Serve para acelerar o processo de criação/execução da sua coroutine

### Chamando qual Dispatchers

```kotlin

lifecycleScope.launch(Dispatchers.IO){
  userDao = roomDB_userDetails.userDao()
}
```

### Dispatchers - Cuidado !!!

```kotlin

viewModelScope.launch(Dispatcher.IO){
  val result = apiCall()
  _myLiveData.value = result
}
```

<img src=".assets/173.jpg">

### Dispatchers - Cuidado !!! - Solução

```kotlin

viewModleScope.launch(Dispatcher.IO){
  val result = apiCall()
  _myLiveData.postValue(result)
}
```
