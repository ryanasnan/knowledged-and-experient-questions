# Knowledge & Experience Questions

Pertanyaan :
Apa tantangan terbesar yang pernah anda temui saat membuat web application dan

Jawab :
Optimasi load image
Pada waktu itu saya mendapatkan task untuk bisa mempercepat load halaman pada web ugm.ac.id.
Masalahnya sebenarnya cukup simple, halaman yang dimuat menjadi sangat lama dikarenakan tiap artikel/post terdapat file image yang cukup besar (> 400kb).
Dan belum terdapat penanganan load small image/thumbnail, sehingga pada beberapa child post memuat gambar yang sama besarnya.

Untuk langkah pertama saya memodifikasi penanganan penyimpanan dan penyuntingan yang terdapat adanya upload image.
setiap file image yang akan diupload (dari post, imageslider, album, dll) akan dipecah dalam beberapa file.

Yang terbanyak dipecah adalah dari imageslider (gambar slider depan) yang dipecah menjadi 4 ukuran yaitu (1140X500, 975X560, 730X420, 148X85),
sedangkan untuk post atau album hanya dipecah menjadi 2 ukuran.
kesemua file yang akan diupload akan disimpan dengan format "[namafile]--[ukuranresolusi].[namaekstensi]" misal "rektor--1140X500.jpg".

Dan dalam setiap mekanisme pengambilan data (fetch) yang mana terdapat "show image" akan diberikan adanya suffix dengan tambahan ukuran resolusi tersebut, serta pemberian lokasi path di file tersebut.
misal dalam fetch imageslider dari original_img_name "rektor.jpg" maka hasilnya adalah "galleries/cropSlider/rektor--1140X500.jpg"
misal dalam fetch post dari original_img_name "rektor.jpg", maka hasilnya adalah "galleries/crop/rektor--730X500.jpg"

Sampai saat tersebut maka setelah membuat post atau imageslider akan lebih terbantu dalam load halamannya karena file tidak sebesar aslinya.

Namun kemudian terdapat adanya masalah baru yaitu : bagaimana dengan gambar yang lama/lawas yang mana belum dipecah ?
Tentu tidak mungkin dengan mengedit tiap post atau imageslider satu persatu agar file pecahan tercipta.

Untuk penanganan ini maka bisa dibuat dari manajemen url.
aplikasi tersebut menggunakan framework yii sehingga tiap pengaksesan url dapat dibuat manajemennya.

Misalkan terdapat pengaksesan seperti "galleries/crop/gambarlama--1140x500px.jpg" untuk load image

url manager yang dapat dibuat untuk menangani pengaksesan tersebut adalah "galleries/crop/<id:[0-9]+>--<size:.+>" dan akan diakses melalui renderImagesController.
Dalam controller tersebut akan mengecek apakah sudah terdapat adanya file jika belum, maka akan dibuatkan.

Cukup simple, namun sangat membantu apalagi dengan framework yang cukup lawas.

penanganan url:

```sh
'galleries/crop/<id:[0-9]+>--<size:.+>' => 'parseImage/Crop',
'galleries/cropSlider/<name:.+>--<size:.+>' => 'renderImages/cropSlider',
```

akses post image:
```sh
public function actionCrop($id,$size)
{
	//memecah width dan height dari string size yang dimasukkan
	preg_match_all('!\d+!', $size, $numSize);
	$width = $numSize[0][0];
	$height = $numSize[0][1];

	//penanganan jika memasukkan size selain yang ditentukan
	$arrSize = array($width,$height);
	$defaultSize = array(array("1140","500"),array("730","420"),array("975","560"),array("148","85"));

	if (!in_array($arrSize, $defaultSize)) {
		$width = "730";
		$height = "420";
	}

	// get image_file_name by $id

	if($data) {
		if(file_exists('galleries/'.$data['image_file_folder'].'/'.$data['image_file_name'])){
		   $file='galleries/'.$data['image_file_folder'].'/'.$data['image_file_name'];
		}else{
		   $file='images/header.png';
		}
		$imageId = $id;
		$extensionImageFile = pathinfo($data['image_file_name'], PATHINFO_EXTENSION);
	}else{
	   $file='images/header.png';
	   $imageId = 'header';
	   $extensionImageFile = 'png';
	}
```

akses imageslider

```sh
public function actionCropSlider($name, $size)
{
	$fileExtenstion = pathinfo($size, PATHINFO_EXTENSION);
	$oriFilePath='images/slider/'.$name.'.'.$fileExtenstion;

	$width = "1140";
	$height = "500";

	Yii::import( 'application.vendors.phpthumb.*');
	require_once 'ThumbLib.inc.php';

	$thumb = PhpThumbFactory::create($oriFilePath);
	$thumb->adaptiveResize($width,$height);
	$thumb->save("galleries/cropSlider/".$name.'--'.$width.'x'.$height.'px'.'.'.$fileExtenstion);
	$thumb->show();
}
```

fetch tiap data post
```sh
foreach ($data as $key => $value) {
    $imageName = pathinfo($value['imgbld_image'], PATHINFO_FILENAME);
    $imageExtension = pathinfo($value['imgbld_image'], PATHINFO_EXTENSION);

    $slider[$key]['url'] = $lang == 'en' ? $value['imgbld_url_en'] : $value['imgbld_url_id'];
    $slider[$key]['src'] = 'galleries/cropSlider/' . $imageName . '--1140x500px' . '.' . $imageExtension;
    //...
}
```
---

Pertanyaan :
Bagaimana penerapan clean code pada project anda?

Jawab :
Terdapat 4 poin yang selalu saya soroti yaitu :
  1. Membuat adanya pembagian pembagian mekanisme kerja menjadi instruksi instruksi yang kecil yang mana hanya fokus dalam menyelesaikan satu permasalahan, sehingga ketika ada penambahan fitur tidak perlu mengubah hampir sebagian besar code yang telah ada.
Semisal adalah mengambil suatu nilai/value yang mana nilai tersebut juga terdapat adanya pengolahan tersendiri, maka dalam kasus tersebut, perlu adanya pembuatan function tersendiri dalam pemrosesan nilai.
 
  2. DRY (Dont Repeat Yourself) atau jangan mengulangi suatu mekanisme yang sebenarnya sama atau telah ada.
Setiap adanya kesamaan mekanisme pada beberapa bagian kode harus dijadikan menjadi satu.

  3. Pemberian nama pada class, function dan variabel yang sesuai, sehingga mudah untuk dimengerti
Dengan berprinsip pada penggunaan nama yang tepat secara linguistik, maka developer lain akan mudah mengerti, sehingga tidak terlalu sulit untuk di maintenance.
Misalkan penggunaan pronounce yang tepat seperti loadData() yang diberikan pada client/javascript dan getData() yang diberikan pada backend.

  4. Meminimalisir komentar di setiap mekanisme, yang merepotkan adalah jika semisal terdapat adanya perubahan nantinya dan komentar juga tidak diubah maka ini akan menyulitkan programmer.
Kecuali untuk komentar yang mana akan memberikan keterangan rincian pada tiap mekanisme, seperti parameter, return data atau mungkin nama dari pembuat kode(author)

---
Pertanyaan :
Untuk efisiensi pengerjaan project dalam tim, jelaskan bagaimana gitflow yang biasa ?

Jawab :
Selama ini saya dengan tim menggunakan git dengan branch yang terpisah pada setiap developer, misalkan satu developer membuat fork/membuat branch sendiri. Selanjutnya tiap developer akan melakukan push di masing masing branch, kemudian tim leader (CTO) melakukan pull request pada branch master.

---

Pertanyaan :
Apa yang anda ketahui dengan design pattern? Jika pernah menggunakan, jelaskan
design pattern apa saja yang biasanya digunakan untuk menyelesaikan masalah
software engineering di web application.

Jawab :
Design pattern adalah desain pembuatan mekanisme kerja yang memiliki adanya kesamaan pola.
Terdapat beberapa macam dari design pattern seperti Factory pattern, singleton pattern, strategy pattern, dll. 

Selama ini di laravel, saya sendiri sering menggunakan Factory pattern seperti misal dalam penggunaan facade atau repository pattern (penanganan dalam akses database).
Tujuan daripada design pattern tersebut adalah untuk memudahkan dalam modifikasi dari tiap bagian fungsi.
Prinsipnya adalah pengaksesan dari hulu ke hilir seperti digambarkan berikut:
$object->getParentProcess()->getChildProcess

---

Pertanyaan :
Apa anda bersedia ditempatkan onsite di Malang? Jika memang harus remote,
bagaimana pengaturan waktu & availability dalam komunikasi dan kolaborasi
pengerjaan project?

Jawab :
Saya bersedia ditempatkan onsite di Malang selama 2 bulan,

Untuk saat ini, saya sebenarnya masih belum siap untuk bekerja secara remote, dikarenakan masih belum memiliki keahlian dan kecekatan yang mumpuni.

Namun apabila memang diwajibkan untuk remote maka saya akan memiliki alur pengerjaan seperti berikut:

  - Sesi 1 Awal (08.00-08-30)
-> dimulai dengan briefing/diskusi kecil mengenai apa yang akan dikerjakan hari ini.

  - Sesi 2 Pencarian referensi (08.30-09.30)
-> mencari referensi terkait dengan yang akan dikerjakan, pada akhir saya akan memberikan simpulan pada tugas yang akan dibuat, yang mana ini berisikan kesanggupan dalam nantinya mengerjakan atau mungkin terdapat hal yang masih belum jelas
jika semuanya telah siap maka tugas pada hari itu akan dituliskan di dalam trello (kanban card) agar terdapat nota kesepahaman.

  - Sesi 3. Pengerjaan (09.30-16.00)
-> mengerjakan tugas, namun sebisa mungkin akan saya berikan keterangan di tiap pengerjaannya.

  - Sesi 4. Laporan (16.00-17.00)
-> memberikan laporan dari apa yang telah dibuat sehingga dapat di review.
