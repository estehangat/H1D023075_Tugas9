# Tugas 9 - Pertemuan 11

```
Nama      : Daiva Paundra Gevano
NIM       : H1D023075
Shift     : A
Shift KRS : F
```

## 🌟 Fitur Aplikasi

- ✅ Registrasi & Login dengan Token
- ✅ CRUD Produk (Create, Read, Update, Delete)
- ✅ Validasi Form Lengkap
- ✅ Error Handling

---

## 🏗️ Arsitektur Aplikasi

Aplikasi menggunakan **BLoC Pattern** dengan struktur:

```
lib/
├── model/          # Data models
├── bloc/           # Business logic
├── ui/             # User interface
├── helpers/        # API & utilities
└── widget/         # Reusable widgets
```

---

## 📝 Proses Registrasi

**File:** `lib/ui/registrasi_page.dart`

<img height="500" alt="image" src="https://github.com/user-attachments/assets/f3ecf09a-7397-4918-9c25-f23de236683e" />
<img height="500" alt="image" src="https://github.com/user-attachments/assets/4d3cd988-59b1-421f-a2b6-51f7c0658ff8" />

### Fungsi Utama:

**1. Validasi Email:**
```dart
Widget _emailTextField() {
  return TextFormField(
    decoration: const InputDecoration(labelText: "Email"),
    validator: (value) {
      // Regex pattern untuk validasi format email
      Pattern pattern = r'^[a-zA-Z0-9.]+@[a-zA-Z0-9]+\.[a-zA-Z]+';
      RegExp regex = RegExp(pattern.toString());
      // Jika email tidak sesuai format, return pesan error
      if (!regex.hasMatch(value!)) return "Email tidak valid";
      return null;
    },
  );
}
```
**Penjelasan:** Widget ini membuat input field untuk email dengan validasi menggunakan Regular Expression (RegEx). RegEx memeriksa apakah email mengikuti format standar (contoh: user@domain.com). Jika tidak valid, akan menampilkan pesan error di bawah field.

**2. Submit Registrasi:**
```dart
void _submit() {
  // Panggil fungsi registrasi di BLoC dengan data dari form
  RegistrasiBloc.registrasi(
    nama: _namaTextboxController.text,
    email: _emailTextboxController.text,
    password: _passwordTextboxController.text,
  ).then((value) {
    // Jika berhasil, tampilkan dialog sukses
    showDialog(
      context: context,
      builder: (context) => SuccessDialog(
        description: "Registrasi berhasil, silahkan login",
      ),
    );
  });
}
```
**Penjelasan:** Fungsi ini dipanggil saat user menekan tombol "Registrasi". Data dari form (nama, email, password) diambil dari TextEditingController dan dikirim ke RegistrasiBloc. Jika berhasil, akan menampilkan dialog sukses yang memberi tahu user untuk login.

**File:** `lib/bloc/registrasi_bloc.dart`

```dart
class RegistrasiBloc {
  static Future<Registrasi> registrasi({
    String? nama, String? email, String? password,
  }) async {
    // Siapkan data dalam format Map untuk dikirim ke API
    var body = {"nama": nama, "email": email, "password": password};
    // Kirim POST request ke endpoint registrasi
    var response = await Api().post(ApiUrl.registrasi, body);
    // Parse response JSON menjadi object Registrasi
    return Registrasi.fromJson(json.decode(response.body));
  }
}
```
**Penjelasan:** BLoC (Business Logic Component) adalah layer yang memisahkan logika bisnis dari UI. Fungsi ini menerima data registrasi, mengemas dalam format JSON, mengirim ke API backend, dan mengembalikan response yang sudah di-parse menjadi object Registrasi.

---

## 🔐 Proses Login

**File:** `lib/ui/login_page.dart`

<img height="500" alt="image" src="https://github.com/user-attachments/assets/7db1ccc5-fd5c-45fc-99e8-7795e2b0472f" />

### Fungsi Utama:

**Submit Login:**
```dart
void _submit() {
  // Panggil fungsi login di BLoC dengan email dan password
  LoginBloc.login(
    email: _emailTextboxController.text,
    password: _passwordTextboxController.text,
  ).then((value) async {
    // Cek apakah response code 200 (berhasil)
    if (value.code == 200) {
      // Simpan token ke SharedPreferences untuk autentikasi selanjutnya
      await UserInfo().setToken(value.token.toString());
      // Simpan userID untuk identifikasi user
      await UserInfo().setUserID(int.parse(value.userID.toString()));
      // Redirect ke halaman produk (pushReplacement menghapus history login)
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (context) => const ProdukPage()),
      );
    }
  });
}
```
**Penjelasan:** Fungsi ini menangani proses login. Setelah user memasukkan email dan password, data dikirim ke API melalui LoginBloc. Jika login berhasil (code 200), aplikasi menyimpan token dan userID ke SharedPreferences. Token ini sangat penting karena akan digunakan untuk autentikasi setiap request ke API selanjutnya. Setelah itu, user diarahkan ke halaman produk menggunakan `pushReplacement` sehingga user tidak bisa kembali ke halaman login dengan tombol back.

**File:** `lib/bloc/login_bloc.dart`

```dart
class LoginBloc {
  static Future<Login> login({String? email, String? password}) async {
    // Siapkan data login dalam format Map
    var body = {"email": email, "password": password};
    // Kirim POST request ke endpoint login
    var response = await Api().post(ApiUrl.login, body);
    // Parse response JSON menjadi object Login (berisi token, userID, dll)
    return Login.fromJson(json.decode(response.body));
  }
}
```
**Penjelasan:** BLoC untuk login menerima email dan password, mengirimkannya ke API backend, dan mengembalikan object Login yang berisi token autentikasi, userID, dan informasi lainnya. Token ini akan digunakan untuk mengakses endpoint yang dilindungi (protected endpoints).

**File:** `lib/helpers/user_info.dart`

```dart
// Simpan Token ke SharedPreferences
Future setToken(String value) async {
  final pref = await SharedPreferences.getInstance();
  return pref.setString('token', value);
}

// Ambil Token dari SharedPreferences
Future<String?> getToken() async {
  final pref = await SharedPreferences.getInstance();
  return pref.getString('token');
}
```
**Penjelasan:** SharedPreferences adalah penyimpanan lokal di device yang memungkinkan data tetap tersimpan meskipun aplikasi ditutup. Fungsi `setToken` menyimpan token autentikasi, sedangkan `getToken` mengambil token yang tersimpan. Token ini akan disertakan di setiap HTTP request untuk membuktikan bahwa user sudah login.

---

## 📦 Proses CRUD Produk

### **1. Read - Menampilkan List Produk**

**File:** `lib/ui/produk_page.dart`

<img height="500" alt="image" src="https://github.com/user-attachments/assets/0f6a2bf8-44ab-49e5-b880-7f1781f2386b" />

```dart
body: FutureBuilder<List>(
  // Panggil fungsi untuk mengambil data produk dari API
  future: ProdukBloc.getProduks(),
  builder: (context, snapshot) {
    // Jika data sudah tersedia, tampilkan list produk
    return snapshot.hasData
      ? ListProduk(list: snapshot.data)
      // Jika belum, tampilkan loading indicator
      : const Center(child: CircularProgressIndicator());
  },
),
```
**Penjelasan:** FutureBuilder adalah widget Flutter yang menangani operasi asynchronous. Widget ini memanggil `getProduks()` untuk fetch data dari API. Saat data masih diambil, menampilkan CircularProgressIndicator (loading spinner). Setelah data tersedia, ditampilkan dalam bentuk list melalui widget ListProduk.

**File:** `lib/bloc/produk_bloc.dart`

```dart
static Future<List<Produk>> getProduks() async {
  // Kirim GET request ke API untuk mengambil semua produk
  var response = await Api().get(ApiUrl.listProduk);
  // Decode response JSON
  var jsonObj = json.decode(response.body);
  // Ambil array data dari response
  List<dynamic> listProduk = jsonObj['data'];
  // Siapkan list kosong untuk menampung object Produk
  List<Produk> produks = [];
  // Loop setiap item dan convert ke object Produk
  for (int i = 0; i < listProduk.length; i++) {
    produks.add(Produk.fromJson(listProduk[i]));
  }
  return produks;
}
```
**Penjelasan:** Fungsi ini mengambil semua data produk dari backend API. Response JSON yang diterima di-parse menjadi List object Produk. Proses ini menggunakan pattern parsing JSON ke model untuk memudahkan akses data dengan type-safe.

### **2. Create - Tambah Produk**

**File:** `lib/ui/produk_form.dart`

<img height="500" alt="image" src="https://github.com/user-attachments/assets/ea48e48c-dd4a-4e98-ba7b-c7c592cb779a" />

```dart
void simpan() {
  // Buat object Produk baru dengan id null (karena belum ada di database)
  Produk createProduk = Produk(id: null);
  // Isi data produk dari input form
  createProduk.kodeProduk = _kodeProdukTextboxController.text;
  createProduk.namaProduk = _namaProdukTextboxController.text;
  createProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);
  
  // Kirim data ke BLoC untuk disimpan ke API
  ProdukBloc.addProduk(produk: createProduk).then((value) {
    // Setelah berhasil, kembali ke halaman list produk
    Navigator.push(context,
      MaterialPageRoute(builder: (context) => const ProdukPage()));
  });
}
```
**Penjelasan:** Fungsi ini membuat produk baru. Data diambil dari TextEditingController (input field), dikemas menjadi object Produk, lalu dikirim ke API melalui BLoC. Setelah berhasil disimpan, user diarahkan kembali ke halaman list produk untuk melihat data yang baru ditambahkan.

**File:** `lib/bloc/produk_bloc.dart`

```dart
static Future addProduk({Produk? produk}) async {
  // Siapkan data dalam format Map untuk dikirim ke API
  var body = {
    "kode_produk": produk!.kodeProduk,
    "nama_produk": produk.namaProduk,
    "harga": produk.hargaProduk.toString(),
  };
  // Kirim POST request ke endpoint create produk
  var response = await Api().post(ApiUrl.createProduk, body);
  // Return status dari response (success/fail)
  return json.decode(response.body)['status'];
}
```
**Penjelasan:** BLoC untuk create produk mengubah object Produk menjadi Map (key-value pairs), lalu mengirimkannya ke API backend menggunakan POST request. API akan menyimpan data ke database dan mengembalikan status operasi.

### **3. Update - Edit Produk**

**File:** `lib/ui/produk_form.dart`

<img height="500" alt="image" src="https://github.com/user-attachments/assets/f28a4057-3212-4391-a266-a2edb856dbc2" />

```dart
void ubah() {
  // Buat object Produk dengan ID yang sama (untuk update existing data)
  Produk updateProduk = Produk(id: widget.produk!.id!);
  // Update data produk dengan nilai baru dari form
  updateProduk.kodeProduk = _kodeProdukTextboxController.text;
  updateProduk.namaProduk = _namaProdukTextboxController.text;
  updateProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);
  
  // Kirim data ke BLoC untuk update di API
  ProdukBloc.updateProduk(produk: updateProduk).then((value) {
    // Setelah berhasil, kembali ke halaman list produk
    Navigator.push(context,
      MaterialPageRoute(builder: (context) => const ProdukPage()));
  });
}
```
**Penjelasan:** Fungsi update mirip dengan create, tetapi menggunakan ID produk yang sudah ada. Ini memberitahu backend bahwa kita ingin mengupdate data existing, bukan membuat data baru. Data baru dari form menggantikan data lama di database.

**File:** `lib/bloc/produk_bloc.dart`

```dart
static Future updateProduk({required Produk produk}) async {
  // Buat URL dengan ID produk untuk endpoint update
  String apiUrl = ApiUrl.updateProduk(int.parse(produk.id!));
  // Siapkan data yang akan diupdate
  var body = {
    "kode_produk": produk.kodeProduk,
    "nama_produk": produk.namaProduk,
    "harga": produk.hargaProduk.toString(),
  };
  // Kirim PUT request dengan body JSON-encoded
  var response = await Api().put(apiUrl, jsonEncode(body));
  // Return status dari response
  return json.decode(response.body)['status'];
}
```
**Penjelasan:** Update menggunakan HTTP method PUT dan menyertakan ID produk di URL. Data di-encode menjadi JSON string menggunakan `jsonEncode()` karena PUT request biasanya mengharapkan content-type application/json. Backend akan mencari produk dengan ID tersebut dan mengupdate field-nya dengan nilai baru.

### **4. Delete - Hapus Produk**

**File:** `lib/ui/produk_detail.dart`

<img height="500" alt="image" src="https://github.com/user-attachments/assets/76989dab-3a50-4adf-b5ac-36a3934b8288" />

```dart
void confirmHapus() {
  // Buat dialog konfirmasi sebelum menghapus
  AlertDialog alertDialog = AlertDialog(
    content: const Text("Yakin ingin menghapus data ini?"),
    actions: [
      OutlinedButton(
        child: const Text("Ya"),
        onPressed: () {
          // Jika user pilih "Ya", kirim request hapus ke API
          ProdukBloc.deleteProduk(id: int.parse(widget.produk!.id!))
            .then((value) {
              // Setelah berhasil dihapus, kembali ke halaman list
              Navigator.push(context,
                MaterialPageRoute(builder: (context) => const ProdukPage()));
            });
        },
      ),
      OutlinedButton(
        child: const Text("Batal"),
        // Jika user pilih "Batal", tutup dialog saja
        onPressed: () => Navigator.pop(context),
      ),
    ],
  );
  // Tampilkan dialog konfirmasi
  showDialog(context: context, builder: (context) => alertDialog);
}
```
**Penjelasan:** Delete adalah operasi yang tidak bisa di-undo, maka penting untuk menampilkan konfirmasi terlebih dahulu. AlertDialog memberikan 2 pilihan: "Ya" untuk melanjutkan penghapusan, atau "Batal" untuk membatalkan. Hanya jika user memilih "Ya", data akan dihapus dari database.

**File:** `lib/bloc/produk_bloc.dart`

```dart
static Future<bool> deleteProduk({int? id}) async {
  // Buat URL dengan ID produk yang akan dihapus
  String apiUrl = ApiUrl.deleteProduk(id!);
  // Kirim DELETE request ke API
  var response = await Api().delete(apiUrl);
  // Parse response
  var jsonObj = json.decode(response.body);
  // Return true/false apakah berhasil dihapus
  return jsonObj['data'];
}
```
**Penjelasan:** Delete menggunakan HTTP method DELETE dengan ID produk di URL. Backend akan mencari produk dengan ID tersebut dan menghapusnya dari database. Response berisi boolean yang menunjukkan apakah operasi berhasil atau gagal.

### **5. API Helper dengan Token**

**File:** `lib/helpers/api.dart`

```dart
// GET Request - Untuk mengambil data
Future<dynamic> get(dynamic url) async {
  // Ambil token dari storage lokal
  var token = await UserInfo().getToken();
  // Kirim GET request dengan token di header untuk autentikasi
  final response = await http.get(Uri.parse(url),
    headers: {HttpHeaders.authorizationHeader: "Bearer $token"});
  // Return response setelah validasi status code
  return _returnResponse(response);
}

// POST Request - Untuk membuat data baru
Future<dynamic> post(dynamic url, dynamic data) async {
  var token = await UserInfo().getToken();
  // Kirim POST request dengan body data dan token
  final response = await http.post(Uri.parse(url), body: data,
    headers: {HttpHeaders.authorizationHeader: "Bearer $token"});
  return _returnResponse(response);
}

// PUT Request - Untuk update data existing
Future<dynamic> put(dynamic url, dynamic data) async {
  var token = await UserInfo().getToken();
  // PUT request butuh Content-Type: application/json
  final response = await http.put(Uri.parse(url), body: data,
    headers: {
      HttpHeaders.authorizationHeader: "Bearer $token",
      HttpHeaders.contentTypeHeader: "application/json"
    });
  return _returnResponse(response);
}

// DELETE Request - Untuk hapus data
Future<dynamic> delete(dynamic url) async {
  var token = await UserInfo().getToken();
  // Kirim DELETE request dengan token untuk autentikasi
  final response = await http.delete(Uri.parse(url),
    headers: {HttpHeaders.authorizationHeader: "Bearer $token"});
  return _returnResponse(response);
}
```
**Penjelasan:** API Helper adalah class yang menangani semua HTTP request ke backend. Setiap method (GET, POST, PUT, DELETE) mengikuti prinsip yang sama:
1. **Ambil token** dari SharedPreferences untuk autentikasi
2. **Sertakan token di header** dengan format "Bearer [token]" - ini membuktikan bahwa user sudah login
3. **Kirim request** sesuai HTTP method yang sesuai
4. **Validasi response** melalui `_returnResponse()` yang memeriksa status code

Token sangat penting karena backend API menggunakan token-based authentication untuk melindungi endpoint dari akses yang tidak sah. Tanpa token yang valid, request akan ditolak dengan status 401 (Unauthorized).

---

## 🚪 Proses Logout

**File:** `lib/ui/produk_page.dart`

<img height="500" alt="image" src="https://github.com/user-attachments/assets/34d1b20d-2291-43bf-b31f-a4b1140ab04a" />

```dart
drawer: Drawer(
  child: ListView(
    children: [
      ListTile(
        title: const Text('Logout'),
        trailing: const Icon(Icons.logout),
        onTap: () async {
          // Panggil fungsi logout untuk hapus data session
          await LogoutBloc.logout().then((value) {
            // Redirect ke halaman login dan hapus semua history navigasi
            // (route) => false berarti hapus semua route dari stack
            Navigator.pushAndRemoveUntil(
              context,
              MaterialPageRoute(builder: (context) => LoginPage()),
              (route) => false, // Hapus semua route history
            );
          });
        },
      ),
    ],
  ),
),
```
**Penjelasan:** Drawer adalah menu samping yang bisa dibuka dengan swipe dari kiri atau tap icon hamburger. Menu logout dipasang di drawer agar mudah diakses dari mana saja. Saat logout diklik, aplikasi menghapus semua data session (token dan userID) dan membawa user kembali ke halaman login. Parameter `(route) => false` pada `pushAndRemoveUntil` memastikan semua halaman sebelumnya dihapus dari navigation stack, sehingga user tidak bisa kembali ke halaman produk dengan tombol back.

**File:** `lib/bloc/logout_bloc.dart`

```dart
class LogoutBloc {
  static Future logout() async {
    // Panggil fungsi logout dari UserInfo untuk hapus session data
    await UserInfo().logout();
  }
}
```
**Penjelasan:** LogoutBloc adalah wrapper sederhana yang memanggil fungsi logout dari UserInfo. Pemisahan ini mengikuti prinsip BLoC pattern di mana semua logika bisnis dihandle di layer BLoC.

**File:** `lib/helpers/user_info.dart`

```dart
Future logout() async {
  // Akses SharedPreferences
  final SharedPreferences pref = await SharedPreferences.getInstance();
  // Hapus token dengan set string kosong
  pref.setString('token', '');
  // Hapus userID dengan set ke 0
  pref.setInt('userID', 0);
}
```
**Penjelasan:** Logout menghapus token dan userID dari SharedPreferences. Dengan menghapus token, aplikasi tidak bisa lagi mengakses API endpoint yang dilindungi, sehingga user harus login kembali. Ini adalah mekanisme keamanan yang penting untuk melindungi data user.

---

## 📡 API Endpoints

**File:** `lib/helpers/api_url.dart`

```dart
class ApiUrl {
  // Base URL adalah alamat utama backend API
  static const String baseUrl = 'http://localhost/tokokita/public';
  
  // Endpoint untuk registrasi user baru
  static const String registrasi = '$baseUrl/registrasi';
  
  // Endpoint untuk login dan mendapatkan token
  static const String login = '$baseUrl/login';
  
  // Endpoint untuk mengambil list semua produk (GET)
  static const String listProduk = '$baseUrl/produk';
  
  // Endpoint untuk membuat produk baru (POST)
  static const String createProduk = '$baseUrl/produk';
  
  // Function untuk generate endpoint update dengan ID produk (PUT)
  static String updateProduk(int id) => '$baseUrl/produk/$id/update';
  
  // Function untuk generate endpoint detail produk dengan ID (GET)
  static String showProduk(int id) => '$baseUrl/produk/$id';
  
  // Function untuk generate endpoint delete dengan ID produk (DELETE)
  static String deleteProduk(int id) => '$baseUrl/produk/$id';
}
```
**Penjelasan:** ApiUrl adalah class yang menyimpan semua endpoint API dalam satu tempat (centralized configuration). Ini memudahkan maintenance - jika base URL berubah, kita hanya perlu mengubah satu tempat. 

- **Static const** digunakan untuk endpoint yang tidak berubah
- **Static function** digunakan untuk endpoint yang memerlukan parameter dinamis (seperti ID)
- Endpoint yang sama (`$baseUrl/produk`) bisa digunakan untuk operasi berbeda tergantung HTTP method:
  - GET → listProduk (ambil semua data)
  - POST → createProduk (buat data baru)
  
Format endpoint mengikuti standard RESTful API:
- `/produk` → Collection (list/create)
- `/produk/{id}` → Single resource (show/update/delete)

---

## 📦 Dependencies

```yaml
dependencies:
  flutter:
    sdk: flutter
  # HTTP client untuk melakukan request ke API (GET, POST, PUT, DELETE)
  http: ^0.13.5
  # SharedPreferences untuk menyimpan data lokal (token, userID)
  shared_preferences: ^2.0.15
```

**Penjelasan Dependencies:**

1. **flutter:** Framework utama untuk membangun aplikasi mobile
2. **http (^0.13.5):** Package untuk melakukan HTTP request ke backend API. Menyediakan method get(), post(), put(), delete() yang mudah digunakan
3. **shared_preferences (^2.0.15):** Package untuk menyimpan data key-value secara persistent di device. Digunakan untuk menyimpan token autentikasi dan userID agar user tetap login meskipun aplikasi ditutup

Symbol `^` (caret) berarti package manager akan menginstall versi yang compatible (contoh: ^0.13.5 akan menginstall versi 0.13.x terbaru, tapi tidak 0.14.0)
