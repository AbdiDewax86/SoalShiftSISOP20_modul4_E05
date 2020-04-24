# SoalShiftSISOP20_modul4_E05
## Soal 1
#### Encrypt dan Decrypt versi 1:
```
//fungsi untuk decrypt
char* Decrypt(char dec[100]){
    char caesar[] = "9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO";
    int z, i, j;

    for (i=0; dec[i]!='\0'; i++){
        for (j=0; j<strlen(caesar); j++)
        {
            if(dec[i] == caesar[j]){
                break;
            }
        }
        z = (j-10)%87;
        if(z < 0){
            z = z + strlen(caesar);
        }
        if(dec[i] == '.' && (strlen(dec)-i)<5)break;
        else if(dec[i] != '/'){
            dec[i] = caesar[z];
        }else{
            dec[i] = '/';
        }
    }
    return dec;
}

//fungsi untuk encrypt
char* Encrypt(char enc[100]){
    char caesar[] = "9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO";
    int i, j;

    for (i=0; enc[i]!='\0'; i++){
        for (j=0; j<strlen(caesar); j++)
        {
            if(enc[i] == caesar[j]){
                break;
            }
        }
        if(enc[i] == '.' && (strlen(enc)-i)<5)break;
        else if(enc[i] != '/'){
            enc[i] = caesar[(j+10)%87];
        }else{
            enc[i] = '/';
        }
    }
    return enc;
}
```
Cara kerja fungsi encrypt ialah, dengan mengecek diposisi manakah posisi huruf teks[i] pada huruf caesar (anggap di posisi j), kemudian menggantikan huruf pada teks[i] dengan huruf caesar yang berada pada j+10.
Fungsi decrypt ialah fungsi untuk mengembalikan nama yang telah terenkripsi menjadi seperti semula, cara kerjanya ialah dengan mengecek di posisi manakah huruf teks[i] yang telah terenkripsi pada huruf caesar, kemudian dikembalikan lagi dengan cara mengubahnya dengan huruf caesar yang berada pada j-10.

Misal kan ada file bernama “kelincilucu.jpg” dalam directory FOTO_PENTING, dan key yang dipakai adalah 10
“encv1_rahasia/FOTO_PENTING/kelincilucu.jpg” => “encv1_rahasia/ULlL@u]AlZA(/g7D.|_.Da_a.jpg

Untuk membuatnya, pada fungsi xmp_readdir, kita harus mengecek apakah path tersebut memiliki parent dengan substring nama "encv1_" atau "/encv1_". Jika benar, maka kita harus mengeknripsi seluruh child pathnya. Pengecekan ini juga berlaku untuk fungsi read,remove,getattr dll.

```
//fungsi untuk mengambil nama path dari folder terenkripsi
char *checkEncrypt(char fpath[100],const char *path)
{
    int i,j;
    char rev[100],fname[1000];
        memset(rev,'\0',100);
        memset(fname,'\0',1000);
        for (i = 0; i<strlen(path); i++)
        {
            if (path[i]=='/')
            {
                break;
            }
            rev[i] = path[i];
        }
        sprintf(fname,"%s",rev);
        memset(rev,'\0',100);
        //printf ("i=%d dan rev = %s\n",i,rev);
        j=0;
        if (i!=strlen(path))
        {
            while (1){
                rev[j] = path[i];
                i++;
                j++;
                if(i==strlen(path))
                {
                    break;
                }
            }
            //printf ("masuk dan rev = %s\n",rev);
            Encrypt(rev);
            //Decrypt(rev);
            strcat(fname,rev);
            sprintf(fpath, "%s%s", dirpath, fname);
            printf ("pathattr1 = %s\n",fpath);
        }
        else
        {
            sprintf(fpath, "%s%s", dirpath, fname);
            printf ("pathattr11 = %s\n",fpath);
        }
    return fpath;
}

//fungsi untuk mengambil nama path dari folder terenkripsi
//dengan '/' pada path[0]
char *checkEncryptslash(char fpath[100],const char *path)
{
    int i,j;
    char rev[100],fname[1000];
        memset(rev,'\0',100);
        memset(fname,'\0',1000);
        for (i = 0; i<strlen(path); i++)
        {
            if ((path[i]=='/' && i!=0) || path[i]=='\0')
            {
                break;
            }
            rev[i] = path[i];
        }
        //mengambil path folder enkripsi
        sprintf(fname,"%s",rev);
        memset(rev,'\0',100);
        j=0;

        //jika terdapat foler/file pada folder enkripsi
        //maka nama path diambil
        if (i!=strlen(path))
        {
            while (1){
                rev[j] = path[i];
                i++;
                j++;
                if(i==strlen(path))
                {
                    break;
                }
            }
            //mengenkripsi nama path didalam folder enkripsi
            Encrypt(rev);
            //menggabungkan fullpath folder enkripsi
            strcat(fname,rev);
            //menggabungkan fullpath dengan directory
            sprintf(fpath, "%s%s", dirpath, fname);
        }
        else
        {
            //jika tidak terdapat file/folder didalam folder terenkripsi
            sprintf(fpath, "%s%s", dirpath, fname);
        }
    return fpath;
}
```
Fungsi ini akan mengambil dan mengenkripsi child path dari folder yang terenkripsi. Contoh :
encv1_XXX/encrypted/encrypted 
Kemudian kembali digabung dengan directorypath nya.

## Soal 4
Log system:

- Sebuah berkas nantinya akan terbentuk bernama "fs.log" di direktori *home* pengguna (/home/[user]/fs.log) yang berguna menyimpan daftar perintah system call yang telah dijalankan.
- Agar nantinya pencatatan lebih rapi dan terstruktur, log akan dibagi menjadi beberapa level yaitu INFO dan WARNING.
- Untuk log level WARNING, merupakan pencatatan log untuk syscall rmdir dan unlink.
- Sisanya, akan dicatat dengan level INFO.
Format untuk logging yaitu:
[LEVEL]::[yy][mm][dd]-[HH]:[MM]:[SS]::[CMD]::[DESC ...]

```
//fungsi untuk membuat log
void createlog(char process[100],char fpath[100])
{
    char text[200];
    FILE *fp = fopen("/home/dimasadh/fs.log","a");
    time_t t = time(NULL);
    struct tm tm = *localtime(&t);
    if (strcmp(process,"unlink")==0)
    {
        sprintf(text, "WARNING::%04d%02d%02d-%02d:%02d:%02d::UNLINK::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,fpath);
    }
    else if (strcmp(process,"mkdir")==0)
    {
        sprintf(text, "INFO::%04d%02d%02d-%02d:%02d:%02d::MKDIR::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,fpath);
    }
    else if (strcmp(process,"rmdir")==0)
    {
        sprintf(text, "WARNING::%04d%02d%02d-%02d:%02d:%02d::RMDIR::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,fpath);
    }
    for (int i = 0; text[i] != '\0'; i++) {
            fputc(text[i], fp);
    }
    fclose (fp);
}

//fungsi untuk membuat log khusus proses rename
void createlogrename(char from[100], char to[100])
{
    FILE *fp = fopen("/home/dimasadh/fs.log","a");
    time_t t = time(NULL);
    struct tm tm = *localtime(&t);
    char text[200];
    sprintf(text, "INFO::%04d%02d%02d-%02d:%02d:%02d::RENAME::%s::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,from,to);
    for (int i = 0; text[i] != '\0'; i++) {
            fputc(text[i], fp);
    }
    fclose(fp);
}
```
Cara kerjanya ialah dengan membuka file fs.log, kemudian mengisinya sesuai dengan permintaan. Khusus untuk log rename, fungsi akan meminta path awal dan path akhir untuk mengisinya kedalam file fs.log
