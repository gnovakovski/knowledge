O Java tem por padrão 4 tipos de referências: forte(Strong), suave(Soft), fraca(Weak) e fantasma(Phantom).

Uma referência forte é a referência normal no Java. Sempre que criamos um novo objeto, uma referência forte é criada por padrão. Exemplo:

``
MyObject object = new MyObject();
``

Este objeto é fortemente acessível - o que significa que ele pode ser acessado por uma cadeia de referências fortes.
Isso impedirá o Garbage Collector de pegar este objeto e destrui-lo.
Mas, aqui está um exemplo de como isso pode jogar contra nós.


    public class MainActivity extends Activity {
      @Override
      protected void onCreate(Bundle savedInstanceState) {   
          super.onCreate(savedInstanceState);
          setContentView(R.layout.main);
          new MyAsyncTask().execute();
      }
    
    private class MyAsyncTask extends AsyncTask {
        @Override
        protected Object doInBackground(Object[] params) {
            return doSomeStuff();
        }
        private Object doSomeStuff() {
            //do something to get result
            return new MyObject();
        } 
    }
    }
    
A AsyncTask vai ser criada e executada junto ao onCreate(). Mas aí está o problema: a classe interna vai ser acessada 
pela classe externa durante todo o ciclo de vida.
O que acontece quando a Activity é destruida? A AsyncTask está segurando a referência para a Activity,
e a Activity não será coletada pelo GarbageCollector. Isso é o que chamamos de um **Memory Leak**.


**WeakReference:** Uma referência fraca(Weak) é uma referência que não é forte o suficiente para manter o objeto em memória. 
Se tentarmos definir se o objeto é fortemente referênciado e isso aconteceu através de uma WeakRefgerences, o GarbageCollector irá destruir o Objeto. Aqui está o mesmo código de antes, mas feito com uma Weak Reference.

    public class MainActivity extends Activity {
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          new MyAsyncTask(this).execute();
      }
    private static class MyAsyncTask extends AsyncTask {
        private WeakReference<MainActivity> mainActivity;    
        
        public MyAsyncTask(MainActivity mainActivity) {   
            this.mainActivity = new WeakReference<>(mainActivity);            
        }
        @Override
        protected Object doInBackground(Object[] params) {
            return doSomeStuff();
        }
        private Object doSomeStuff() {
            //do something to get result
            return new Object();
        }
        @Override
        protected void onPostExecute(Object object) {
            super.onPostExecute(object);
            if (mainActivity.get() != null){
                //adapt contents
            }
        }
    }
    }

Repare no seguinte trecho:

 ``private WeakReference<MainActivity> mainActivity;``
 
 Desta forma, quando a activity for destruida, nenhum Memory Leak será criado.
 
**SoftReference:** pense em uma SoftReference como uma WeakReference mais forte. Enquanto uma WeakReference será coletada imediatamente, uma SoftReference implorará ao GarbageCollector para permanecer na memória, a menos que não haja outra opção. Os algoritmos do Garbage Collector são realmente emocionantes e algo em que você pode mergulhar por horas e horas sem se cansar. Mas basicamente, eles dizem “Eu sempre recuperarei a WeakReference. Se o objeto for um SoftReference, vou decidir o que fazer com base nas condições do ecossistema ”. Isso torna um SoftReference muito útil para a implementação de um cache: desde que haja memória suficiente, não precisamos nos preocupar em remover objetos manualmente. Se você quiser ver um exemplo em ação, pode verificar este [exemplo de um cache implementado com SoftReference](https://peters-andoird-blog.blogspot.com/2012/05/softreference-cache.html).
 
**PhantomReference:** Um Objeto que só foi referenciado por meio de um PhantomReference pode ser coletado sempre que o Coletor de Lixo desejar.


Referência: https://medium.com/google-developer-experts/finally-understanding-how-references-work-in-android-and-java-26a0d9c92f83
