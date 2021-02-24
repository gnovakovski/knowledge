### Async task
Async tasks por padrão, a partir do android 3.0(api 11), executam uma após a outra.
Se quiser executar em paralelo, deve chamar: new verifyOpenOrderTask(this, view).executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR);

https://developer.android.com/reference/android/os/AsyncTask.html#order-of-execution
