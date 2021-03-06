### 检查日期是否是指定的格式
```php
function checkDatetime($str, $format="Y-m-d H:i:s"){
    $unixTime  = strtotime($str);
    $checkDate = date($format, $unixTime);

    return $checkDate == $str;
}
```

### 互换两个变量的值

```php
$x = 10;
$y = 11;

$x = $x + $y;
$y = $x - $y;
$x = $x - $y;

var_dump($x, $y);  // int(11) int(10)
```

### 获取当前页面的URL

```php
/**
 * 获取当前页面完整URL地址
 */
function get_url() {
    // 协议
    $protocol = isset($_SERVER['SERVER_PORT']) && $_SERVER['SERVER_PORT'] == '443'
        ? 'https://' : 'http://';

    $host = isset($_SERVER['HTTP_HOST']) ? $_SERVER['HTTP_HOST'] : '';

    // 脚本名称
    $php_self = $_SERVER['PHP_SELF'] ? $_SERVER['PHP_SELF'] : $_SERVER['SCRIPT_NAME'];

    // PATH_INFO
    $path_info = isset($_SERVER['PATH_INFO']) ? $_SERVER['PATH_INFO'] : '';

    $relate_url = isset($_SERVER['REQUEST_URI'])
        ? $_SERVER['REQUEST_URI']
        : $php_self . (isset($_SERVER['QUERY_STRING'])
            ? '?' . $_SERVER['QUERY_STRING'] : $path_info);

    return $protocol . $host . $relate_url;
}
```


### 解析url中的query参数为数组

```php
/**
 * 获取url中的?后的查询参数
 *
 * @param string $url url地址
 *
 * @return array
 */
function convertUrlQuery($url)
{
    $arr    = parse_url($url);
    $query  = explode('&', $arr['query']);

    $params = array();
    foreach ($query as $param) {
        $item = explode('=', $param);
        $params[$item[0]] = $item[1];
    }

    return $params;
}
```

> 参考：[PHP解析URL并得到URL中的参数](http://blog.csdn.net/wide288/article/details/17712989)

### 文件下载
对于浏览器不能直接打开的文件，比如 .zip、.exe、.xsl 等，可以直接使用一个 a 元素来指向这个文件资源，点击就能直接下载。而对于 .jpg 等文件，如果直接点击链接，就是在浏览器中打开了，而不是提示我们下载保存。

我们是通过 Header 请求头来发送文件下载信息，指定下载的是附件，下载后的文件名，content-length 来指定文件的大小，然后通过 readfile 函数来读取文件内容而实现文件下载：

```php
<?php 
 
$filename = $_GET['filename'];
header('content-disposition:attachment;filename='. basename($filename));
header('content-length:'. filesize($filename));
 
readfile($filename);
```

### 截取字符串，并补全省略号
截取字符串中指定长度的子串，如果截取长度小于字符串总长度，则添加省略号，否则不添加。而且可以从左侧或右侧截取，还能设置字符串的编码格式。

> 需确保 PHP 支持`mb_substr`和`mb_strlen`函数。

```php
/**
 * 截取字符串,并根据情况添加省略号
 *
 * @param string $text   要进行截取的字符串
 * @param int    $length 截取的长度
 * @param string $encode 字符串编码
 * 
 * @return string
 */
function subtext($text, $length, $encode = 'utf8')
{
    $text_len = mb_strlen($text, $encode);
    
    // 如果字符串长度大于截取长度,则进行截取和补全省略号
    if ($text_len > abs($length)) {
        // 如果$length为正数则表示从左往右截取,最后在右侧补全省略号
        // 否则表示从右往左截取,并在左侧补全省略号
        if ($length >= 0) {
            return mb_substr($text, 0, $length, $encode) . '...';
        } else {
            return '...' . mb_substr($text, $length, $text_len, $encode);
        }
    }
    
    return $text;
}
```

### 接收 base64 图片数据

```php
function base64save($img, array $types = []) {
    // 图片路径地址
    $baseDir = 'upload/base64/' . date('Ymd') . '/';
    if (!is_dir($baseDir)) {
        mkdir($baseDir, 077, true);
    }
    
    $types  = count($types) ? types : ['jpg', 'gif', 'png', 'jpeg'];
    $img    = str_replace(['_', '-'], ['/', '+'], $img);
    $b64img = substr($img, 0, 100);
    $b64reg = '/^(data:\s*image\/(\w+);base64,)/';
    
    if (preg_match(b64reg, $b64img, $matches)) {
        if (!in_array($matches[2], $types)) {
            return array(
                'status' => 1,
                'info'   => '图片格式不正确，只支持'.implode(', ', types).'哦！',
                'url'    => ''
            );
        }
        
        $img  = str_replace($matches[1], '', $img);
        $img  = base64_decode($img);
        $path = $baseDir . md5(date('YmdHis') . rand(1000, 9999)) . '.' . $matches[2];
        file_put_contents($path, $img);
        
        return ['status' => 0, 'info' => '保存图片成功', 'url' => $path];
    }
    
    return ['status' => 2, 'info' => '请选择要上传的图片'];
}
```

### PHPExcel 类库的使用

```php
// 导出 Excel
header('Content-Type: application/vnd.ms-excel');
header(sprintf('Content-Disposition: attachment;filename="%s.xls"',$filename));
header('Cache-Control: max-age=0');

// 导入 Excel
if (isset($_FILES["file"]) && 0 == $_FILES["file"]["error"]) {
  $fileType = array("application/vnd.ms-excel", "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", "application/kset","application/octet-stream");  //文件格式
            
  if (in_array($_FILES["file"]["type"], $fileType) && $_FILES["file"]["size"] <= 2 * 1000 * 1000) {
    $file = $_FILES['file']['tmp_name'];
    require_once BASEPATH . '/libraries/phpexcel/PHPExcel.php';
    $excelReader = new PHPExcel_Reader_Excel2007();

    if (!$excelReader->canRead($file)) {
        $excelReader = new PHPExcel_Reader_Excel5();
    }

    $sheet = $excelReader->load($file)->getSheet(0); //sheet1操作
    $excelCont = array(
        'highestCol' => $sheet->getHighestColumn(), //列
        'highestRow' => $sheet->getHighestRow(), //行
        'highestColumnIndex' => PHPExcel_Cell::columnIndexFromString($sheet->getHighestColumn()) // 几列
    );
  }
}
```


