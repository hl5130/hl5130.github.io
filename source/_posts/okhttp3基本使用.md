---
title: okhttp3基本使用
date: 2019-04-08 14:42:59
tags: okHttp3
categories: Android
---

# 简介
从Android4.4开始，系统内置了OkHttp。 okHttp3 有重大改版，例如 FormEncodingBuilder 类没有了，取而代之的是 FormBody。
## 基本步骤
- 创建 OkHttpClient
- 创建 Request
- 创建 Call
- 调用 Call 的 enqueue() 或者 execute()

# GET请求
```
       // 1、Request
       Request.Builder builder = new Request.Builder();
       builder.url("http://www.hong1969.com");
       builder.method("GET",null);
       Request request = builder.build();

       // 2、 HttpClient
       OkHttpClient client = new OkHttpClient();

       // 3、Call
       Call call = client.newCall(request);

       // 4、异步请求
       call.enqueue(new Callback() {
           @Override
           public void onFailure(@NonNull Call call, @NonNull IOException e) {
               e.printStackTrace();
           }

           @Override
           public void onResponse(@NonNull Call call, @NonNull Response response) throws IOException {
               ResponseBody responseBody = response.body();
               if (null != responseBody) {
                   String string = responseBody.string();
                   Log.e(LOG_TAG,"返回结果："+string);
               } else {
                   Log.e(LOG_TAG,"ResponseBody is null");
               }
           }
       });
```

# POST请求
```
        // 1、FormBody
        FormBody.Builder formBodyBuilder = new FormBody.Builder();
        formBodyBuilder.add("username","admin");
        formBodyBuilder.add("password","123456");
        FormBody formBody = formBodyBuilder.build();

        // 2、Request
        Request.Builder builder = new Request.Builder();
        builder.url("http://distributionapi.1000fun.com/login/userlogin");
        builder.method("POST",formBody);
        Request request = builder.build();

        // 3、 HttpClient
        OkHttpClient client = new OkHttpClient();

        // 4、Call
        Call call = client.newCall(request);

        // 5、异步请求
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NonNull Call call, @NonNull IOException e) {
                e.printStackTrace();
            }

            @Override
            public void onResponse(@NonNull Call call, @NonNull Response response) throws IOException {
                ResponseBody responseBody = response.body();
                if (null != responseBody) {
                    String string = responseBody.string();
                    Log.e(LOG_TAG,"返回结果："+string);
                } else {
                    Log.e(LOG_TAG,"ResponseBody is null");
                }
            }
        });
```

# 异步上传文件
- 上传文件也是``POST``请求，并使用``RequestBody``设置请求体
- 上传文件的格式通过``MediaType``来设置。例如``MediaType mediaType = MediaType.parse("text/x-markdown; charset=utf-8")``就是用来设置上传txt文件

```
        // 1、文件路径
        String filePath = "";
        if (Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)){
            filePath = Environment.getExternalStorageDirectory().getAbsolutePath();
        } else {
            return;
        }
        // 2、文件
        File file = new File(filePath,"wangshu.txt");

        // 3、RequestBody，上传的是txt格式的文件
        MediaType mediaType = MediaType.parse("text/x-markdown; charset=utf-8");
        RequestBody requestBody = RequestBody.create(mediaType, file);

        // 4、Request
        Request.Builder requestBuilder = new Request.Builder();
        requestBuilder.url("https://api.github.com/markdown/raw")
                .post(requestBody)
                .build();
        Request request = requestBuilder.build();

        // 5、HttpClient
        OkHttpClient client = new OkHttpClient();

        // 6、Call
        Call call = client.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NonNull Call call, @NonNull IOException e) {
                e.printStackTrace();
                if (e instanceof FileNotFoundException){
                    Log.e(LOG_TAG,"该文件不存在");
                }
            }

            @Override
            public void onResponse(@NonNull Call call, @NonNull Response response) throws IOException {
                ResponseBody responseBody = response.body();
                if (null != responseBody) {
                    String string = responseBody.string();
                    Log.e(LOG_TAG,"返回结果："+string);
                } else {
                    Log.e(LOG_TAG,"ResponseBody is null");
                }
            }
        });
```

# 异步下载文件
- 可以不用设置请求方式
- 通过IO流对``Response``进行处理

```
        // 1、Request
        String url = "https://www.baidu.com/img/bd_logo1.png";
        Request request = new Request.Builder().url(url).build();

        // 2、OkHttpClient 设置
        OkHttpClient client = new OkHttpClient();

        // 3、Call
        Call call = client.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NonNull Call call, @NonNull IOException e) {
                e.printStackTrace();
                if (e instanceof FileNotFoundException){
                    Log.e(LOG_TAG,"该文件不存在");
                }
            }

            @Override
            public void onResponse(@NonNull Call call, @NonNull Response response) {
                ResponseBody responseBody = response.body();
                if (null != responseBody) {
                    InputStream inputStream = responseBody.byteStream();
                    String filePath;
                    try {
                        if (Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)){
                            filePath = Environment.getExternalStorageDirectory().getAbsolutePath();
                        } else {
                            filePath = context.getFilesDir().getAbsolutePath();
                        }
                        File file = new File(filePath,"wangshu.jpg");
                        FileOutputStream fileOutputStream = new FileOutputStream(file);
                        byte[] buffer = new byte[1024];
                        int len = 0;
                        while ( (len = inputStream.read(buffer)) != -1){
                            fileOutputStream.write(buffer,0,len);
                        }
                        fileOutputStream.flush();

                    } catch (IOException e){
                        e.printStackTrace();
                    }

                } else {
                    Log.e(LOG_TAG,"ResponseBody is null");
                }
            }
        });
```

# 异步上传Multipart文件
## 使用场景
在上传文件的时候，需要同时上传参数
## 基本步骤
- 使用``MultipartBody.Builder``创建``RequestBody``
- 创建``Request``，并包含``RequestBody``
- 其他步骤不变

```
        // 1、设置MediaType
        final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");

        // 2、设置 RequestBody
        RequestBody requestBody = new MultipartBody.Builder()
                .setType(MultipartBody.FORM)
                .addFormDataPart("title","wangshu")
                // 第一个参数是key值，第二个参数上传文件的名字，第三个参数是上传的文件
                .addFormDataPart("image","wangshu.png", RequestBody.create(MEDIA_TYPE_PNG, new File("/storage/emulated/0/wangshu.png")))
                .build();

        // 3、设置 Request
        Request request = new Request.Builder()
                .url("https://.....")
                .post(requestBody)
                .build();

        // 4、设置 OkHttpClient
        OkHttpClient okHttpClient = new OkHttpClient();

        // 5、 Call
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NonNull Call call, @NonNull IOException e) {

            }

            @Override
            public void onResponse(@NonNull Call call, @NonNull Response response) throws IOException {

            }
        });
```
