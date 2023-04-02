<p align="center">
  <img src="https://portmanat.az/media/last/style/img/logo.png" alt="Logo">
  <h3 align="center">Portmanat checkout</h3>
</p>


## Məzmun
* [Ümumi məlumat](#about)
* [Endpointlər](#endpoints)
* [PHP kod](#phpcode)
* [Java kod](#javacode)
* [Python kod](#pythoncode)

<a name="about"/>

## Ümumi məlumat

Portmanat checkout ilə xidmətiniz üçün bir düymə ilə ödəniş qəbul edə, çoxfunksiyalı interfeys ilə müxtəlif kriteriyalar üzrə statistikasına baxa və onu fayl şəklində saxlaya bilərsiz. Portmanat checkout saytınızda və mobil tətbiqinizdə quraşdırılması çox asandır. 
Bu səhifədə ümumi anlayışlar əks etdirilmişdir.<br/><br/>
Biznes proses aşağıdakı ardıcıllıqlar davam edir
1. Tranzaksiya yaradılması - bu addımda müştəri haqqında məlumatlar (`service_id`, `order_id`, `amount`,`secret_key`) və təhlükəsizlik üçün `token` dəyəri `/checkout/create` endpointinə sorğu göndərilir. Api ünvandan əldə etdiyiniz `transaction_id` (property) dəyəri ilə birgə tranzaksiya yaradılır. Tranzaksiya yaratdıqdan sonra müştərini ödəniş səhifəsinə `redirect_url` (property sorğuda qayıdır) yönləndirilir.
2. Ödəniş səhifəsi - Bu proses tamamilə Portmanat checkout səhifəsində aparılır.
3. Nəticə - Müştəri ödəniş etdikdən sonra sizin partner paneldə xidmət yaradarkən qeyd etdiyiniz `redirect_url` ünvanına yönləndirilir. Bu linkin sonuna `?transaction_id={ilk mərhələdəki tranzaksiya}` birləşdirilir
4. Tranzaksiyanın yoxlanılması - səhifənizə qayıdan müştərinin `transaction_id` məlumatı ilə `/status/{transaction_id}` endpointi ilə ödənişin statusunu yoxlaya bilərsiz

<br/>
Aşağıda istifadə olunmuş dəyişənlər haqqında məlumat tapa bilərsiz:<br/><br/>

| Dəyişən adı  | Tipi | Dəyərlər |
|-------------|-------------|-------------|
| payment_type  | integer  | 1 - hesab, 2 - kod, 3 - plastik kart|
| service_id  | integer  | xidmət id, partner panelə bax |
| order_id  | string  | sifarişin unikal kodu, siz təyin edirsiz |
| amount  | numeric  | xidmətin qiyməti, siz təyin edirsiz |
| token  | string  | yaradılma üsulu aşağıda göstərilib |
| secret_key  | string  | məxfi söz, partner panelə bax |
<br/>

`token` dəyərinin yaradılma qaydası:<br/>
`service_id`, `order_id`, `amount` və `secret_key` müvafiq ardıcıllıqla birləşdirilir(Concatenation) və MD5 hash yaradılır.

PHP kod nümunəsi :
``` 
$token = md5($service_id.$order_id.$amount.$secret_key); 
```
Python kod nümunəsi :
``` 
plain_text = "{}{}{}{}".format(service_id, order_id, amount, secret_key).encode('utf-8')
token = hashlib.md5(plain_text).hexdigest()
```
<a name="endpoints"/>

## Endpointlər

| Endpoint  | Metod | Parametrlər | Məlumat
|-------------|-------------|-------------|-------------|
| /checkout/create  | POST  | payment_type, service_id, <br/>order_id, amount, token|Tranzaksiya yaradılması|
| /checkout/status/{transaction_id}  | GET  | token|Tranzaksiya yoxlanılması|

<a name="phpcode"/>

## PHP kod

Tranzaksiya yaradılması

```
$service_id = 4;
$order_id = 1;
$amount = 1000;
$secret_key = '123456';
$payment_type = 0;
$token = md5($service_id . $order_id . $amount . $secret_key);

$curl = curl_init();
curl_setopt_array($curl, array(
    CURLOPT_URL => 'http://127.0.0.1:8000/en/checkout/create/',
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => '',
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => true,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => 'POST',
    CURLOPT_POSTFIELDS => array(
        'payment_type' => $payment_type,
        'service_id' => $service_id,
        'order_id' => $order_id,
        'amount' => $amount,
        'token' => $token
    ),
    CURLOPT_HTTPHEADER => array(
        'Accept: application/json',
    ),
));
$response = curl_exec($curl);
curl_close($curl);
```


Tranzaksiyanın yoxlanılması

```
$service_id = 4;
$order_id = 1;
$amount = 1000;
$secret_key = '123456';
$token = md5($service_id . $order_id . $amount . $secret_key);

$curl = curl_init();
curl_setopt_array($curl, array(
    CURLOPT_URL => 'http://127.0.0.1:8000/en/checkout/status/e6554945-8caf-472c-9f6c-2640f33efb1d?token=4bfb74cf2d435ebf4109acbbed8ff52c',
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => '',
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => true,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => 'GET',
    CURLOPT_HTTPHEADER => array(
        'Accept: application/json',
    ),
));
$response = curl_exec($curl);
curl_close($curl);
```


<a name="javacode"/>

## Java kod

Tranzaksiya yaradılması

```
int service_id = 4;
int order_id = 1;
float amount = 1000;
String secret_key = "123456";
int payment_type = 0;
String plain_text = String.format("%d%d%f%s", service_id, order_id, amount, secret_key);
MessageDigest md5 = MessageDigest.getInstance("MD5");
md5.update(StandardCharsets.UTF_8.encode(plain_text));
String token = String.format("%032x", new BigInteger(1, md5.digest()));

String url = "http://127.0.0.1:8000/en/checkout/create/";

OkHttpClient client = new OkHttpClient().newBuilder().build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = new MultipartBody.Builder().setType(MultipartBody.FORM)
  .addFormDataPart("payment_type", payment_type)
  .addFormDataPart("service_id", service_id)
  .addFormDataPart("order_id", order_id)
  .addFormDataPart("amount", amount)
  .addFormDataPart("token", token)
  .build();
Request request = new Request.Builder()
  .url(url)
  .method("POST", body)
  .addHeader("Content-Type", "application/json")
  .addHeader("Accept", "application/json")
  .build();
Response response = client.newCall(request).execute();
```

Tranzaksiyanın yoxlanılması

```
int service_id = 4;
int order_id = 1;
float amount = 1000;
String secret_key = "123456";
String plain_text = String.format("%d%d%f%s", service_id, order_id, amount, secret_key);
MessageDigest md5 = MessageDigest.getInstance("MD5");
md5.update(StandardCharsets.UTF_8.encode(plain_text));
String token = String.format("%032x", new BigInteger(1, md5.digest()));

String url = "http://127.0.0.1:8000/en/checkout/status/e6554945-8caf-472c-9f6c-2640f33efb1d?token=4bfb74cf2d435ebf4109acbbed8ff52c";

OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("text/plain");
RequestBody body = RequestBody.create(mediaType, "");
Request request = new Request.Builder()
  .url(url)
  .method("GET", body)
   .addHeader("Content-Type", "application/json")
  .addHeader("Accept", "application/json")
  .build();
Response response = client.newCall(request).execute();

```

<a name="pythoncode"/>

## Python kod

Tranzaksiya yaradılması

```
service_id = 4
order_id = 1
amount = 1000
secret_key = '123456'
payment_type = 0
plain_text = "{}{}{}{}".format(service_id, order_id, amount, secret_key).encode('utf-8')
token = hashlib.md5(plain_text).hexdigest()

import requests

url = 'http://127.0.0.1:8000/en/checkout/create/'
payload={
  'payment_type': payment_type,
  'service_id': service_id,
  'order_id': order_id,
  'amount': amount,
  'token': token
}
headers = {
  'Content-Type': 'application/x-www-form-urlencoded'
}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)

```


Tranzaksiyanın yoxlanılması

```
service_id = 4
order_id = 1
amount = 1000
secret_key = '123456'
plain_text = "{}{}{}{}".format(service_id, order_id, amount, secret_key).encode('utf-8')
token = hashlib.md5(plain_text).hexdigest()

import requests

url = "http://127.0.0.1:8000/en/checkout/status/e6554945-8caf-472c-9f6c-2640f33efb1d?token=4bfb74cf2d435ebf4109acbbed8ff52c"
payload={}
headers = {
  'Accept': 'application/json'
}
response = requests.request("GET", url, headers=headers, data=payload)
print(response.text)
```
