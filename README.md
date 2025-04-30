## Soal 1
Trabowo dan sahabatnya, Peddy, sedang menikmati malam minggu di rumah sambil mencari film seru untuk ditonton. Mereka menemukan sebuah file ZIP yang berisi gambar-gambar poster film yang sangat menarik. File tersebut dapat diunduh dari Google Drive. Karena penasaran dengan film-film tersebut, mereka memutuskan untuk membuat sistem otomatis guna mengelola semua file tersebut secara terstruktur dan efisien. Berikut adalah tugas yang harus dikerjakan untuk mewujudkan sistem tersebut:

### Bagian A
Trabowo langsung mendownload file ZIP tersebut dan menyimpannya di penyimpanan lokal komputernya. Namun, karena file tersebut dalam bentuk ZIP, Trabowo perlu melakukan unzip agar dapat melihat daftar film-film seru yang ada di dalamnya.

```c
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    int status;

    pid_t child1 = fork();
    if(child1 == 0){
        char *argv[] = {
            "wget",
            "--no-check-certificate",
            "https://drive.google.com/uc?export=download&id=1nP5kjCi9ReDk5ILgnM7UCnrQwFH67Z9B",
            "-O",
            "film.zip",
            NULL
        };
        execv("/usr/bin/wget", argv);
    }
    wait(&status);

    pid_t child2 = fork();
    if(child2 == 0){
        char *argv[] = {"unzip", "film.zip", NULL};
        execv("/usr/bin/unzip", argv);
    }
    wait(&status);

    pid_t child3 = fork();
    if(child3 == 0){
        char *argv[] = {"rm", "film.zip", NULL};
        execv("/bin/rm", argv);
    }
    wait(&status);
}
```

- menggunakan `fork()` untuk membuat variabel child1, child2, dan child3.
- child1 merupakan proses yang mendownload film.zip dengan menggunakan `wget`
- child2 merupakan proses yang meng-unzip file film.zip dengan `unzip`, secara efektif membuat folder baru yang berbeda dengan film.zip
- child3 merupakan proses yang me-remove file film.zip dengan `rm`
- ketiganya menggunakan `*argv` untuk menyimpan command dan `execv` untuk mengeksekusi command.
- `wait()` digunakan agar prosesnya berjalan secara bergantian.
  ![image](https://github.com/user-attachments/assets/2e628c96-bd39-46ab-b253-bbaa0f7efd1f)
  ![image](https://github.com/user-attachments/assets/f7d1f745-4d81-457d-8b86-2d963cfbf9ec)



### Bagian B
Setelah berhasil melakukan unzip, Trabowo iseng melakukan pemilihan secara acak/random pada gambar-gambar film tersebut untuk menentukan film pertama yang akan dia tonton malam ini.

```c
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <sys/wait.h>

int main(){
    int fd[2];
    pid_t child_id;
    char *files[70];
    int count = 0;

    if(pipe(fd) < 0) exit(1);

    child_id = fork();

    if(child_id == 0){
        close(fd[0]);
        dup2(fd[1], STDOUT_FILENO);
        close(fd[1]);

        char *argv[] = {"ls", "film", NULL};
        execv("/bin/ls", argv);
    } 
    else{
        close(fd[1]);
        char temp[1000];
        ssize_t maxbyte = read(fd[0], temp, sizeof(temp) - 1); //biar temp gk besar2 banget
        temp[maxbyte] = '\0';

        char *point = strtok(temp, "\n");
        while(point != NULL){
            files[count] = strdup(point);
            count++;
            point = strtok(NULL, "\n");
        }

        close(fd[0]);
        wait(NULL); // tunggu child selesai

        srand(time(NULL));
        int index = rand() % count;
        printf("Film for Trabowo & Peddy: '%s'\n", files[index]);
        
        for(int i = 0; i < count; i++){
            free(files[i]);
        }
    }

    return 0;
}
```

- mendeklarasikan `pipe(fd)` sekaligus mengecek apakah pipe nya berhasil.
- `fork()` child_id, masuk ke child process. Menggunakan `dup2(fd[1], STDOUT_FILENO)` untuk mengindikasikan bahwa apa yang muncul di terminal akan masuk ke writing end dari sebuah pipe.
- masih di child process, meng-`exec` ls, yang akan dimasukkan ke writing end.
- masuk ke parent process (setelah else), menutup writing end dengan `closefd[1]`. Mencari ukuran optimal untuk sebuah array buffer `temp` dengan `maxbyte` (mencari besar dari reading pipe `fd[0]`) dan memberi karakter NULL di index terakhir (sekaligus mentransfer hasil ls yang dii child process ke array temp).
- menggunakan `strtok` untuk memparsing per file dan memasukan file-file nya ke dalam array `files`. Variabel `count` diincrement untuk mencari tau ada berapa file didalamnya.
- `wait(NULL)` untuk menunggu proses-proses selesai.
- menggunakan `srand()` dan `rand()` untuk mengambil angka secara acak. Kemudian mengeprint nama file berdasarkan index yang sudah diperoleh secara acak.
- menghapus data dari array files[] dengan `free()`.
  ![image](https://github.com/user-attachments/assets/d09c0792-7365-4912-aa5e-5c477bac266c)


### Bagian C
Trabowo membuat 3 folder: FilmHorror, FilmAnimasi, FilmDrama. Kemudian, dengan 2 proses (dari atas dan dari bawah) memindahkan film-film ke folder berdasarkan genre. Lalu buatlah file “recap.txt” yang menyimpan log setiap kali mereka selesai melakukan task. Setelah memindahkan semua film, Trabowo dan Peddy juga perlu menghitung jumlah film dalam setiap kategori dan menuliskannya dalam file total.txt

```c
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <sys/wait.h>

void writelog(const char *filename, const char *genre, const char *orang) {
    FILE *log = fopen("log.txt", "a");
    if(log == NULL) return;

    time_t t = time(NULL);
    struct tm tm = *localtime(&t);

    fprintf(log, "[%02d-%02d-%d %02d:%02d:%02d] %s: %s telah dipindahkan ke %s\n",
            tm.tm_mday, tm.tm_mon + 1, tm.tm_year + 1900,
            tm.tm_hour, tm.tm_min, tm.tm_sec, orang, filename, genre);
    fclose(log);
}

void movefile(const char *filename, const char *folder){
    char source[256], desti[256];
    snprintf(source, sizeof(source), "film/%s", filename);
    snprintf(desti, sizeof(desti), "film/%s/%s", folder, filename);

    char *argv[] = {"mv", source, desti, NULL};
    execv("/bin/mv", argv);
}

const char* findgenre(const char *filename) {
    const char *tempp = strrchr(filename, '_');
    if(!tempp) return NULL;
    if(strstr(tempp, "horror.jpg")) return "FilmHorror";
    if(strstr(tempp, "animasi.jpg")) return "FilmAnimasi";
    if(strstr(tempp, "drama.jpg")) return "FilmDrama";
    return NULL;
}

int main(){
    if(fork() == 0){
        char *argv[] = {"mkdir", "-p", "film/FilmHorror", "film/FilmAnimasi", "film/FilmDrama", NULL};
        execv("/bin/mkdir", argv);
    }
    wait(NULL);

    int fd[2];
    if(pipe(fd) < 0) exit(1);

    pid_t child = fork();
    if(child == 0) {
        close(fd[0]);
        dup2(fd[1], STDOUT_FILENO);
        close(fd[1]);
        char *argv[] = {"ls", "film", NULL};
        execv("/bin/ls", argv);
    }

    close(fd[1]);
    char temp[4000];
    ssize_t maxbyte = read(fd[0], temp, sizeof(temp) - 1);
    temp[maxbyte] = '\0';
    close(fd[0]);
    wait(NULL);

    char *files[70];
    int count = 0;
    char *point = strtok(temp, "\n");
    while(point != NULL){
        files[count++] = strdup(point);
        point = strtok(NULL, "\n");
    }

    int pipep[2], pipet[2];
    pipe(pipep);
    pipe(pipet);

    pid_t peddyd = fork();
    if(peddyd == 0){
        close(pipep[0]);
        int countp[3] = {0};
        int batas = count/2;
        for(int i = 0; i < batas; i++){
            const char *genre = findgenre(files[i]);
            if(!genre) continue;

            if(strcmp(genre, "FilmHorror") == 0) countp[0]++;
            if(strcmp(genre, "FilmAnimasi") == 0) countp[1]++;
            if(strcmp(genre, "FilmDrama") == 0) countp[2]++;
            pid_t p = fork();
            if(p == 0){
                writelog(files[i], genre, "Peddy");
                movefile(files[i], genre);
            }
            wait(NULL);
        }
        write(pipep[1], countp, sizeof(countp));
        close(pipep[1]);
        exit(0);
    }

    pid_t trabowod = fork();
    if(trabowod == 0){
        close(pipet[0]);
        int countt[3] = {0};
        int batas2 = count/2;
        for (int i = count - 1; i >= batas2; i--) {
            const char *genre = findgenre(files[i]);
            if(!genre) continue;

            if(strcmp(genre, "FilmHorror") == 0) countt[0]++;
            if(strcmp(genre, "FilmAnimasi") == 0) countt[1]++;
            if(strcmp(genre, "FilmDrama") == 0) countt[2]++;
            pid_t p = fork();
            if(p == 0){
                writelog(files[i], genre, "Trabowo");
                movefile(files[i], genre);
            }
            wait(NULL);
        }
        write(pipet[1], countt, sizeof(countt));
        close(pipet[1]);
        exit(0);
    }

    wait(NULL);
    wait(NULL);

    close(pipep[1]);
    close(pipet[1]);

    int fromp[3] = {0}, fromt[3] = {0};
    read(pipep[0], fromp, sizeof(fromp));
    read(pipet[0], fromt, sizeof(fromt));

    close(pipep[0]);
    close(pipet[0]);

    int countgenre[3];
    countgenre[0] = fromp[0] + fromt[0];
    countgenre[1] = fromp[1] + fromt[1];
    countgenre[2] = fromp[2] + fromt[2];

    char maxgenre[30]; int maxtemp = 0;
    for(int i = 0; i < 3; i++){
        if(maxtemp < countgenre[i]){
            maxtemp = countgenre[i];
        }
    }
    
    char *gennre[3] = {"horror", "animasi", "drama"};
    for(int i = 0; i < 3; i++){
        if(maxtemp == countgenre[i]){
            strcpy(maxgenre, gennre[i]);
        }
    }
    

    FILE *total = fopen("total.txt", "w");
    fprintf(total, "Jumlah film horror: %d\n", countgenre[0]);
    fprintf(total, "Jumlah film animasi: %d\n", countgenre[1]);
    fprintf(total, "Jumlah film drama: %d\n", countgenre[2]);
    fprintf(total, "Genre dengan jumlah film terbanyak: %s", maxgenre);
    fclose(total);

    return 0;
}
```

- pertama, membuat child proses yang meng-`exec` `mkdir` dan membuat 3 folder yaitu, "FilmHorror", "FilmAnimasi", dan "FilmDrama".
- **kurang lebih sama dengan nomor 2 untuk mengisi array `files[]` dengan data film-film.**
- membuat pipe `pipet` dan `pipep` yang berfungsi untuk mentransfer data dari child process ke parent process mengenai jumlah film per genre.
- membuat process `peddyd` dan `trabowod` dimana peddy memulai dari awal ke tengah sedangkan trabowo dari akhir ke tengah.
- Mekanisme trabowo dan peddy: mencari genre dengan fungsi `findgenre()`, mengincrement `countp` atau `countp` berdasarkan index--genre nya, menulis log dengan fungsi `writelog()` dan memindahkan film ke respective folder dengan `movefile()`. Setelah semuanya selesai, write countt/countp ke pipe masing-masing dan transfer datanya ke parent process.
- Mekanisme findgenre: menggunakan `strrchr` untuk mengambil parsing terakhir berdasarkan underscore, meng-return berdasarkan genre yang ditemukan.
- Mekanisme movefile: menggunakan `snprintf` untuk membuat string berdasarkan `filename` dan folder genre, meng-`exec` `mv` untuk memindahkan file ke folder berdasarkan genre.
- Mekanisme writelog: membuka file `log.txt` dengan `fopen` ("a" menandakan bahwa penulisan file dilanjutkan dari yang ditinggalkan sebelumnya), mendapatkan waktu reallife dengan `time` dan `localtime`, menggunakan `fprintf` untuk menulis kedalam log.txt, menutup file dengan `fclose`.
- kembali ke main process, setelah dikirimkan jumlah film per genre dengan variable `countt` dan `countp`, gunakan `read(pipe[0]` pada masing-masing pipe untuk membaca reading end nya dan simpan dalam `fromp` dan `fromt`. Tambahkan kedua pipe pada indeks yang sama (0 = horror, 1 = animasi, 2 = drama) dan simpan dalam array `countgenre[]`.
- gunakan simple bubble sort dengan kondisi untuk mencari indeks dengan value terbesar.
- buka file `total.txt` dengan `fopen` dan gunakan `fprintf` untuk menuliskan jumlah film per genre, serta genre dengan film terbanyak. fclose().
![Screenshot 2025-04-30 165401](https://github.com/user-attachments/assets/6798781c-73d2-4068-93fe-86409a042cd0)
![Screenshot 2025-04-30 165412](https://github.com/user-attachments/assets/a3305a5e-05dd-4905-aa7d-4cc7f6f842af)
![Screenshot 2025-04-30 155854](https://github.com/user-attachments/assets/f5fbd53a-6c05-4c19-9cce-99a23e43acfe)
![Screenshot 2025-04-30 165346](https://github.com/user-attachments/assets/48bc6e7e-0c80-40fe-9eab-5e2b3258efc9)


### Bagian D
Setelah semua film tertata dengan rapi dan dikelompokkan dalam direktori masing-masing berdasarkan genre, Trabowo ingin mengarsipkan ketiga direktori tersebut ke dalam format ZIP agar tidak memakan terlalu banyak ruang di komputernya.

```c
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdio.h>

int main(){
    pid_t child1 = fork();
    if(child1 == 0){
        char *argv[] = {"zip", "-r", "film.zip", "film/FilmHorror", "film/FilmAnimasi", "film/FilmDrama", NULL};
        execv("/usr/bin/zip", argv);
    }

    pid_t child2 = fork();
    if(child2 == 0){
        char *argv[] = {"rm", "-r", "film", NULL};
        execv("/bin/rm", argv);
    }
}
```
- mirip dengan soal bagian A
- child1 adalah proses yang meng-zip foldernya dengan menggunakan `zip` diikuti dengan nama file yang diinginkan dan ketiga folder yang ingin dizip.
- child2 adalah proses yang meng-remove folder film dengan menggunakan `rm`.
