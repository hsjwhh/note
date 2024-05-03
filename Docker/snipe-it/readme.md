## 注意事项

- 首先，我们需要生成一个应用程序密钥。如果您已经运行了Snipe-IT容器，请记下APP_KEY。在旧版本的Snipe-IT中，它存储在app/config/production/app.php中。运行容器，生成 APP_KEY，整体（带前缀 base64: )传递给环境变量或者放在文件中。详见：https://snipe-it.readme.io/docs/docker
  ```sh
  docker run --rm snipe/snipe-it

  输出结果，应该和下面差不多：
  Please re-run this container with an environment variable $APP_KEY
  An example APP_KEY you could use is: 
  base64:D5oGA+zhFSVA3VwuoZoQ21RAcwBtJv/RGiqOcZ7BUvI=
  ```
  
