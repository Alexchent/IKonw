# php实现下载文件

```php

function download_file($url, $savePath) {
        $ch = curl_init($url);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        curl_setopt($ch, CURLOPT_NOBODY, 0);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $package = curl_exec($ch);
        $httpInfo = curl_getinfo($ch);

        // 自动校准文件类型
        $fileType = $httpInfo["content_type"];
        list(,$type) = explode("/",$fileType);
        list($filename, $ext) = explode(".", $savePath);
        $savePath = $filename .".". $type;

        curl_close($ch);

        if (!$package) {
            return false;
        }

        //创建目录并设置权限
        $basePath = dirname($savePath);
        if (!file_exists($basePath)) {
            @mkdir($basePath, 0777, true);
            @chmod($basePath, 0777);
        }
        // 将文件内容保存到指定路径
        $fp = fopen($savePath, 'w');
        if (!$fp) {
//            echo "无法打开文件进行写入: ". $savePath;
            return false;
        }
        $writeResult = fwrite($fp, $package);
        fclose($fp);

        if (!$writeResult) {
//            echo "文件写入失败";
            return false;
        }

        return true;
    }
```