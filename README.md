# Membuat Cluster MPI

Pada tutorial ini, kita akan memahami langkah-langkah yang diperlukan untuk membuat sebuah cluster MPI di lingkungan Ubuntu 22.04. Dari instalasi perangkat lunak hingga konfigurasi setiap node dalam cluster, tutorial ini akan memberikan panduan yang komprehensif bagi pemula maupun mereka yang ingin memperdalam pemahaman mereka tentang komputasi paralel.

## Persiapan
<ol>
  <li> Siapkan beberapa komputer. Tentukan satu komputer untuk master dan beberapa komputer untuk worker. </li>
  <li> Pastikan seluruh komputer terhubung dalam satu jaringan (misalnya: terhubung dengan Wifi yang sama). </li>
  <li> Update dan upgrade package `sudo apt update && sudo apt upgrade` </li>
  <li> Install beberapa package sebagai utilitas, yaitu `net-tools` dan `vim` (opsional). </li>
</ol>

## Konfigurasi File `/etc/hosts`
### 1. Untuk Master
Buka file `/etc/hosts` menggunakan editor teks seperti vim, vi, nano, atau yang sejenisnya. Selanjutnya, tambahkan beberapa entri IP dan alias untuk setiap komputer. Berikut adalah contohnya.
```bash
10.9.56.77 master
10.9.58.78 worker1
10.9.57.33 worker2
```
### 2. Untuk worker
Sama seperti master, Kemudian tambahkan entri IP dan alias hanya untuk IP master dan IP komputer itu sendiri. Contohnya seperti berikut.
```bash
10.9.56.77 master
10.9.58.78 worker1
```
Contoh:
</br>
![Configurasi Hosts - Master.jpeg](https://github.com/Arditriyudha/Open-MPI/blob/main/Configurasi%20Hosts%20-%20Master.jpeg)
## Buat User Baru
### 1. Buat User
> lakukan di **Master** dan di **worker**
Buat user baru dengan nama yang sama di masing-masing komputer. pada tutorial ini saya menggunakan nama `mpiuser`
```bash
sudo adduser mpiuser
```
### 2. Beri Akses Root
> lakukan di **Master** dan di **worker**
```bash
sudo usermod -aG mpiuser
```
### 3. Masuk ke User yang Telah Dibuat
> lakukan di **Master** dan di **worker**
```bash
su - mpiuser
```
## Konfigurasi SSH
Setelah masuk ke dalam akun pengguna yang telah Anda buat sebelumnya, lakukan konfigurasi SSH dengan mengikuti langkah-langkah berikut:
### 1. Install SSH
> lakukan di **Master** dan di **worker**
```bash
sudo apt install openssh-server
```
anda bisa mencoba untuk menghubungkan komputer dengan SSH
```bash
ssh <nama user>@<host>
# Misal: ssh mpiuser@worker1
```
contoh :
</br>
![Check SSH - Master.jpeg](https://github.com/Arditriyudha/Open-MPI/blob/main/Check%20SSH%20-%20Master.jpeg)

### 2. Buat Key
> lakukan di **Master**
masukkan perintah berikut.
```bash
ssh-keygen -t -rsa
```
Anda akan diminta untuk memberikan beberapa input, namun Anda dapat melewatinya tanpa harus memberikan informasi tambahan.

Kunci SSH akan disimpan di direktori `~/.ssh` dengan nama `id_rsa` untuk kunci privat dan `id_rsa.pub` untuk kunci publik.

### 3. Salin Key Publik ke Worker
> Lakukan di **Master**
Kunci publik harus disalin ke setiap worker dan disimpan dalam file `authorized_keys`. Tindakan ini dapat dilakukan di master menggunakan SSH.
```bash
cd ~/.ssh
cat id_rsa.pub | ssh <nama user>@<host> "mkdir .ssh cat >> .ssh/authorized_keys"
```
Gantilah `<nama pengguna>` dengan nama pengguna yang bersangkutan dan `<host>` dengan alamat host dari worker tersebut. Terapkan perintah ini pada semua worker. Berkas `authorized_keys` akan dibuat di direktori `~/.ssh`.

## Konfigurasi NFS
### 1. Buat Shared Folder
> lakukan di **Master** dan di **worker**
Buatlah direktori dengan nama pilihan Anda, contohnya `cloud`. Disarankan untuk menempatkannya di dalam direktori `~/` atau `~/Desktop`.
```bash
mkdir ~/cloud
```
### 2. Install NFS Server
> Lakukan di **Master**
```bash
sudo apt install nfs-kernel-kernel
```
Contoh:
</br>
![Install NFS-Server - Master.jpeg](https://github.com/Arditriyudha/Open-MPI/blob/main/Install%20NFS-Server%20-%20Master.jpeg)
### 3. Konfigurasi File `/etc/exports`
> Lakukan di **Master**
Buka file `/etc/exports` menggunakan nano (atau gunakan editor teks lainnya jika diinginkan).
tulisakan dengan perintah berikut.
```bash
sudo nano /etc/exports
```
Kemudian tambahkan baris berikut.
```bash
<shared folder> *(rw,sync,no_root_squash,no_subtree_check)
# Misal: /home/mpiuser/cloud *(rw,sync,no_root_squash,no_subtree_check)
```
selanjutnya jalankan perintah berikut.
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```
### Install NFS Client
> Lakukan di **Worker**
```bash
sudo mount <server host>:<shared folder di server> <shared folder di client>
# Misal: sudo mount master:/home/mpiuser/cloud /home/mpiuser/cloud
```
Coba buat file atau folder dalam di shared folder di salah satu komputer. Seharusnya, file atau folder tersebut akan tampil di direktori yang sama di setiap komputer.
Contoh:
</br>

![Install NFS - Worker 2.jpeg](https://github.com/Arditriyudha/Open-MPI/blob/main/Install%20NFS%20-%20Worker%202.jpeg)
## Konfigurasi MPI
### 1. Install MPI
> Lakukan di **Master** dan **worker**
```bash
sudo apt install openmpi-bin libopenmpi-dev
```
Contoh:
</br>
![Install MPI - Worker 2.jpeg](https://github.com/Arditriyudha/Open-MPI/blob/main/Install%20MPI%20-%20Worker%202.jpeg)
### 2. Install Library `mpi4py`
> Lakukan di **Master** dan **worker**
Install `mpi4py` dengan pip. Install package `python3-pip` jika belum memiliki pip.
```bash
sudo apt install python3-pip
pip install mpi4py
```
### 3. Testing MPI
> Lakukan di **Master**
Buat dan edit file `test.py` di shared folder (cloud) dengan menggunakan perintah berikut.
```bash
vim ~/cloud/test.py
```
Kemudian tambahkan beris berikut ke dala file `test.py`
```bash
from mpi4py import MPI
comm = MPI.COMM_WORLD
size = comm.Get_size()
worker = comm.Get_rank()
print("Hello world from worker", str(worker), "of", str(size))
```
Selanjutnya jalankan file `test.py` dengan MPI.
```bash
mpirun -np <jumlah prosesor> -host <daftar host> python3 test.py
# Misal: mpirun -np 3 -host master,worker1,worker2 python3 test.py
```
Output:
```bash
Hello world from worker 1 of 4
Hello world from worker 2 of 4
Hello world from worker 0 of 4
```
Contoh:
Kami juga menggunakan code bs dan mn yang telah di sediakan
</br>
![Run - Master.jpeg](https://github.com/Arditriyudha/Open-MPI/blob/main/Run%20-%20Master.jpeg)

kemudian kami juga mencoba menjalankan beberapa program lain, dan menghasilkan output sebagai berikut.
</br>
![HASIL RUN MASTER.jpeg](https://github.com/Arditriyudha/Open-MPI/blob/main/HASIL%20RUN%20MASTER.jpeg)

Dibuat untuk **memenuhi Tugas Mata Kuliah Pemrosesan Parallel**
**Kelompok 2 Kelas SK5A**
<ol>
  <li> Muhammad Dzaky Khairy (09011182126001)</li>
  <li> Edo Pratama (09011182126007) </li>
  <li> Dhani Saputra (09011182126019)</li>
  <li> Ardi Tri Yudha (09011282126043) </li>
</ol>
