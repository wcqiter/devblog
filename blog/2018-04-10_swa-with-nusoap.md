原本SOAP這種Web service大多使用Java寫成，而基於工作的關係這兒利用php的nusoap library實現SOAP Server和測試用的Client。
為啥要用nusoap，而不是利用原生php的SoapServer和SoapClient呢，原因之一是因為原生庫不支持SOAP with Attachments（SWA），
而因為工作關係需要實現文件上傳的SOAP Service，因此這兒就利用現成的nusoap。
在這個例子裡面Client是用php寫成，因為只是測試用，實際上client和server的開發大多不會是用一個人或是團隊負責，因為利用web service不外乎是需要為同公司或是其他合作夥伴提供資訊交換服務或是利用別處的資訊交換服務，把前端和後段分開。
所以下面的client可以因應條件利用其他的語言，如java寫成。

首先，你需要預備[nusoap](//sourceforge.net/projects/nusoap/)的php檔案。
下載完成後解包，放到項目文件夾裡面。

這個例子中簡略的流程就是：
* 在測試用的html透過AJAX把form中multiple file的formdata傳送到測試用client端。
* Client調用nusoap，建立soap client，然後傳入Soap Server位處的位置。
* Client調用wsdl中註明的方法、變數等資料，向Soap Server發出SOAP Request。
* Server受到SOAP Request後經過處理返還response。

然後我們開始來寫Server，代碼如下：
```
  include('lib/nusoap.php'); // Include nusoap

  $namespace = "fileuploader"; // Namespace of SOAP

  // initialize SOAP Server
  $server = new soap_server;
  $server->configureWSDL("Sample file uploader", $namespace); // Set title
  $server->soap_defencoding='UTF-8'; // Set default encoding
  $server->wsdl->schemaTargetNamespace = $namespace; // Set namespace
  $server->wsdl->addComplexType( // Add complexType of string[]
    'array', 'complexType', 'array', '', '',
    array(),
    array(
      array('ref'=>'SOAP-ENC:arrayType','wsdl:arrayType'=>'xsd:string[]')
    ),
    'xsd:string'
  );
  $server->wsdl->addComplexType( // Add complexType of file[]
    'files', 'complexType', 'array', '', '',
    array(),
    array(
      array('ref'=>'SOAP-ENC:arrayType','wsdl:arrayType'=>'xsd:base64Binary[]')
    ),
    'xsd:base64Binary'
  );
  $server->register( // Add post method
    'Upload', // method name
    array( // method input
      'filename' => 'tns:array',
      'files' => 'tns:files'
    ),
    array( // method output
      'result' => 'xsd:string'
    ),
    $namespace, // namespace
    '', // restriction of xml schema
    'rpc', // XML protocol
    'literal', // encoding method
    'Upload file by this method' // Documentation
  );

  $HTTP_RAW_POST_DATA = isset($GLOBALS['HTTP_RAW_POST_DATA'])
  ? $GLOBALS['HTTP_RAW_POST_DATA'] : file_get_contents("php://input");
  $server->service($HTTP_RAW_POST_DATA);
  exit();

  // Upload method
  function Upload($filename, $files)
  {
    $array = [];
    for($i=0; $i<count($filename); $i++){ // for each file uploaded
      $file_content = base64_decode($files[$i]); // 64 bit based decode to get content of file
      forceFilePutContents("upload" . "/" . $filename[$i], $file_content); // store the file on local directory
      array_push($array, $filename);
    }
    return "Upload success"; // return success message
  }

  // Helper function for putting file on local storage, included directory existence check
  function forceFilePutContents ($filepath, $message){
    try {
      $isInFolder = preg_match("/^(.*)\/([^\/]+)$/", $filepath, $filepathMatches);
      if($isInFolder) {
        $folderName = $filepathMatches[1];
        $fileName = $filepathMatches[2];
        if (!is_dir($folderName)) {
          mkdir($folderName, 0777, true);
        }
      }
      file_put_contents($filepath, $message);
    } catch (Exception $e) {
      echo "ERR: error writing '$message' to '$filepath', ". $e->getMessage();
    }
  }
```

然後是測試用的Client：
```
  include('lib/nusoap.php'); // Include nusoap

  $id = $_POST['id']; // Get id from ajax request

  if ($_FILES['files']["error"] > 0) { // Check files error from ajax request
    echo "Error: " . $_FILES['files']["error"]; // Print error
  } else {
    $filename = []; // array to pass in POST Order method, filename of files uploaded
    $content = []; // array to pass in POST Order method, 64 bit based encoded of files uploaded
    foreach($_FILES as $index => $file) {// For each files
      $fileName     = $file['name']; // File name
      $fileTempName = $file['tmp_name']; // The template file name that the file is stored when uploaded by ajax
      $path = "upload"; // Upload folder
      $filePath = $path . "/" . $file['name'];

      if(!empty($file['error'][$index])) // Check error
      {
        return false;
      }

      if(!is_dir($path)) { // Check directory exist
        mkdir($path);
      }

      if(!empty($fileTempName) && is_uploaded_file($fileTempName)) { // Check file upload ready
        move_uploaded_file($fileTempName, $filePath); // Move file
        array_push($content, base64_encode(file_get_contents($filePath))); // Add file content to array
        array_push($filename, $fileName); // Add file name to array
      }
    }
    $param = array( // $param to inject to soap client call
      'id' => $id,
      'filename' => $filename,
      'upload' => $content
    );

    $client = new nusoap_client("http://localhost:8080/server.php"); // Create a nusoap client
    $err = $client->getError(); // Get error
    if ($err) {
      echo 'Error: ' . $err; // Print error
    }
  }

  $result = $client->call('Upload', $param); // Invoke upload method

  echo (json_encode($result, JSON_PRETTY_PRINT)); // Return result

  }
```
最後是測試用的html：
```
  <html>
  <head>
    <title>基於nusoap實現SOAP with Attachments的SOAP服務</title>
    <!-- Scripts -->
    <script src="//ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script type="text/javascript">
    // Setup ajax first
    $(document).ready(function(){
      $.ajaxSetup({
        headers: {
          'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
      });
    });
    // Ajax request of POST order
    function upload() {
      var formData = new FormData();
      for (var x = 0; x < $("#files").get(0).files.length; x++) {
        formData.append("files" + x, $("#files").get(0).files[x]); // Append each file to form data
      }
      $.ajax({
        method: "POST",
        url: "/client.php",
        contentType: false,
        processData: false,
        data: formData,
        success: function(resdata) {
          console.log(resdata);
          $("#response").html(resdata);
        },
        error: function(result, status, err) {
          console.log(result.responseText);
          console.log(status.responseText);
          console.log(err.Message);
        }
      });
    }
    </script>
  </head>
  <body>
    File:
<input type="file" name="files[]" id="files" multiple>

<input type="submit" value="Upload" onclick="upload();">


<pre id="response"></pre>
</body>
</html>
```
