<h1>Tugas 3 Big Data</h1>

Tugas kali ini adalah mencoba algoritma rekomendasi menggunakan KNIME Workflow, dengan algoritma Collaborative filtering.

<h2>Tools yang dibutuhkan</h2>
<ul>
  <li>KNIME</li>
  <li>MovieLens 20M Dataset (http://files.grouplens.org/datasets/movielens/ml-20m.zip)</li>
</ul>

<h2> CRISP_DM</h2>

![alt](screenshot/workflow.png)

<h3>Business Understanding</h3>
  Pada tugas ini, saya mencoba suatu algoritma untuk merekomendasikan sesuatu berdasarkan kemiripan perilaku user tersebut dengan user lain. Algoritma ini dinamakan Collaborative filtering, dimana didasarkan pada Teknik ALS (alternating least squares).<br>
  Contoh kali ini adalah pemberian rekomendasi film berdasarkan kemiripan antara user satu dengan user lain dalam memberikan peringkat suatu film. Untuk dataset, saya menggunakan dataset MovieLens (20M). Dataset ini berisi kumpulan data film secara umum dengan peringkat film yang telah dinilai oleh banyak pengguna.<br>
<h3>Data Understanding</h3>
Dataset ini berisi sejumlah data berbeda terkait dengan judul dan peringkat film. Saya menggunakan dua file, yaitu rating.csv dan movies.csv <br>
Dataset dalam file ratings.csv berisi 20 juta peringkat film oleh sekitar 130.000 pengguna. Setiap baris berisi movieID (id film), userID(id user), rating(peringkat yang diberikan), timestamp(waktu).<br>
Dataset dalam file movies.csv berisi sekitar 27.000 film. Setiap baris berisi movieID(id film), title(judul film), dan genres(genre dari filmnya).<br>
<h3>Data Preparation</h3>
Pada persiapan data, saya menggunakan dua metode, yaitu file reader dan csv to spark.<br>
Sebelum mengolah data, kita siapkan dulu local spark contextnya,

![alt](screenshot/7.png)

  1. File Reader
  
    Pada data preparation untuk file reader, berikut adalah workflow yang dijalankan.
  
  ![alt](screenshot/0.png)
  
  ![alt](screenshot/1.png)
  
    Pada konfigurasi untuk file reader, saya membaca suatu table yang didownload sebelumnya, yaitu movies.csv.
    
  ![alt](screenshot/2.png)
  
  Table ini akan ditambahkan dua kolom, yaitu timestamps yang diisi 123 dan userID yang diisi 999999. Lalu data dibagi menjadi dua dengan row splitter, data pertama adalah data yang akan dilakukan training, yaitu 20 film yang telah di pilih secara acak, dan data kedua adalah film lainnya yang nantinya akan kita prediksi.
  
  ![alt](screenshot/3.png)
  
  20 film yang telah diberikan rating oleh user dari webview, dimana film ini akan dijadikan bahan training nantinya.
  
  ![alt](screenshot/4.png)
  
  ![alt](screenshot/5.png)
  
  Selanjutnya adalah menjalankan kedua node table to spark, dimana data yang telah berisi rate akan dikirimkan untuk di training, dan data yang tidak ada rate akan diprediksi nantinya, dan akan men-deploy suatu rekomendasi beberapa film untuk user tersebut.
  
  ![alt](screenshot/6.png)
  
  2. CSV to Spark
  
   Pada data preparation untuk spark, berikut adalah workflow yang dijalankan.
    
   ![alt](screenshot/10.png)
    
   Dalam membaca ratings.csv, saya menggunakan node CSV to Spark, dimana ini akan mengconvert CSV langsung menjadi table Spark. Berikut konfigurasinya: 
    
   ![alt](screenshot/8.png)
   
   Selanjutnya adalah dilakukan partisi 80% - 20% pada data csv, dimana 80% digunakan untuk training dan 20% digunakan untuk test.
   
   ![alt](screenshot/9.png)
   
   Setelah itu kita akan lakukan modelling.
    
<h3>Modelling</h3>

![alt](screenshot/12.png)

Untuk modelling, kita harus membuat sebuah ALS model untuk menggunakan algoritma ini. ALS model dibuat dengan spark collaborative filtering learner, dimana akan menghasilkan output berupa matrix factorization model. Output inilah yang akan kita buat untuk evaluasi dan untuk deployment. Sebelum dilakukan collaborative filtering learner, kita concate 80% film original dan 20% film yang telah diberikan rating oleh user. Untuk konfigurasinya:

![alt](screenshot/11.png)

<h3>Evaluation</h3>

Pada evaluation dijalan model seperti yang ada pada model workflow.

![alt](screenshot/13.png)

Jalankan node spark predictor untuk generate prediksi rating dengan test set 20% original film.

![alt](screenshot/15.png)

![alt](screenshot/14.png)

Selanjutnya adalah menjalankan node removal NaN untuk menghapus data NaN atau prediksi yang hilang, setelah itu jalankan spark numeric scorer untuk menghitung kesalahan (error) numerik antara peringkat film original dan peringkat yang diprediksi.

![alt](screenshot/16.png)

Berikut hasil perhitungan errornya:

![alt](screenshot/17.png)

<h3>Deployment</h3>

Selanjutnya adalah tahap terakhir, deploy sebuah model untuk memberikan top 10 prediksi film rekomendasi. Berikut workflownya

![alt](screenshot/18.png)

Lalu jalankan node spark predictor, untuk melakukan prediksi terhadap film yang belum dirating berdasarkan training data model.

![alt](screenshot/19.png)

Kemudian jalankan spark to table untuk memindahkan data dari spark ke dalam tabel, untuk memproses data dalam bentuk table knime. Setelah melakukan convert, kita hilangkan NaN prediction dan lakukan sorting terhadap rating, kita ambil 10 rating terbaik. Setelah itu lakukan join dengan movies.csv.

![alt](screenshot/20.png)

Berikut adalah konfigurasinya:

![alt](screenshot/21.png)

![alt](screenshot/22.png)

![alt](screenshot/23.png)

Hasil rekomendasi film untuk user adalah:

![alt](screenshot/24.png)

Kemudian jalankan node terakhir display recommendation, untuk men-generate suatu webview yang memperlihatkan hasil rekomendasi film tadi. Berikut hasilnya:

![alt](screenshot/25.png)

<h2>Perbandingan waktu</h2>

Perbandingan waktu antara CSV to Spark dan file reader, dimana saya menggunakan contoh dalam membaca rating.csv. Berikut konfigurasi kedua node:

![alt](screenshot/2.1.png)

![alt](screenshot/2.2.png)

Setelah menjalankan kedua node tersebut, saya membuat node timer info, dan inilah hasil dari timer info:

![alt](screenshot/2.3.png)

Dari hasil tersebut, dapat saya simpulkan bahwa node CSV to Spark lebih cepat dalam membaca file dibandingkan file reader.
