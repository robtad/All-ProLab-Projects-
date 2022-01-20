//Muhammad Abdan SYAKURA (200201147)
//Robera Tadesse GOBOSHO (190201141)
#include <stdio.h>
#include <stdlib.h>
int count=0;

typedef struct Kayit
{
    int ogrNo;
    int dersKodu;
    int puan;
    int offset;
}ogrenci;

void counter()
{
    FILE *file;
    if((file = fopen("kayitlar.bin", "rb")) == NULL)
        printf("\nkayitlar.bin DOSYASI BULUNAMADI");
    else
    {
        printf("\n-------------------------\n");
        fseek(file, 0, SEEK_END);
        count = ftell(file)/sizeof(ogrenci);
        fclose(file);
    }
    printf("\n  %d Kayit bulunmaktadir.\n", count);
}

void indexDosyasiOlustur()
{
    ogrenci *s, s1;
    FILE *file, *file2;
    int i=0, j=0;

    if((file = fopen("kayitlar.bin", "rb")) == NULL)
        printf("\nError! kayitlar.bin DOSYASI BULUNAMADI");

    s = (ogrenci*)calloc(count, sizeof(ogrenci));
    if(s == NULL){
        printf("Memory not allocated.\n");
        exit(0);
    }
    while(fread(&s[i], sizeof(ogrenci), 1, file))
    {
        s[i].offset = ftell(file)-sizeof(ogrenci);
        i++;
    }
    fclose(file);

    for(i=0; i<count; i++)
    {
        for(j=i+1; j<count; j++)
        {
            if(s[i].ogrNo > s[j].ogrNo)
            {
                s1 = s[i];
                s[i] = s[j];
                s[j] = s1;
            }
        }
    }
    if((file2 = fopen("index.txt", "w")) == NULL)
        printf("\nError! index.txt DOSYASI OLUSTURULAMADI");
    for(i=0; i<count; i++)
    {
        fprintf(file2, "%d\t%d\n", s[i].ogrNo, s[i].offset);
    }
    fclose(file2);
    printf("\n  index.txt BASARIYLA OLUSTURULMUSTUR!\n");
    free(s);
}

void kayitOlustur()
{
    ogrenci *s;
    FILE *file;
    int n, i;

    printf("Kac tane ogrenci gireceksiniz : ");
    scanf("%d", &n);
    printf("\n");

    s = (ogrenci*)calloc(n, sizeof(ogrenci));
    if(s == NULL){
        printf("Memory not allocated.\n");
        exit(0);
    }

    if((file = fopen("kayitlar.bin", "wb")) == NULL)
    {
        printf("\nError! kayitlar.bin DOSYASI OLUSTURULAMADI");
        exit(1);
    }
    for(i=0;i<n;i++){
        printf("Ogrenci numarayi giriniz : ");
        scanf("%d", &s[i].ogrNo);
        printf("Ders kodunu giriniz : ");
        scanf("%d", &s[i].dersKodu);
        printf("Puan giriniz : ");
        scanf("%d", &s[i].puan);
        printf("\n");
        fwrite(&s[i], sizeof(ogrenci), 1, file);
    }
    fclose(file);
    counter();
    indexDosyasiOlustur();
}

void kayitEkle()
{
    ogrenci *s;
    FILE *file;
    int n, i;

    printf("Kac tane ogrenci gireceksiniz : ");
    scanf("%d", &n);
    printf("\n");

    s = (ogrenci*)calloc(n, sizeof(ogrenci));
    if(s == NULL){
        printf("Memory not allocated.\n");
        exit(0);
    }

    if((file = fopen("kayitlar.bin", "ab")) == NULL)
    {
        printf("Error! kayitlar.bin ACILAMADI");
        exit(1);
    }
    for(i=0;i<n;i++){
        printf("Ogrenci numarayi giriniz : ");
        scanf("%d", &s[i].ogrNo);
        printf("Ders kodunu giriniz : ");
        scanf("%d", &s[i].dersKodu);
        printf("Puan giriniz : ");
        scanf("%d", &s[i].puan);
        printf("\n");
        fwrite(&s[i], sizeof(ogrenci), 1, file);
    }
    fclose(file);
    free(s);
    counter();
    printf("  %d tane kayit basariyla eklendi", n);
    indexDosyasiOlustur();
}

void kayitBul()
{
    ogrenci s1;
    int *ogrNo, *offset, search;
    FILE *file, *file2;

    printf("ARAMAK ICIN OGRENCI NO YAZINIZ : ");
    scanf("%d", &search);

    ogrNo = (int*)calloc(count, sizeof(int));
    offset = (int*)calloc(count, sizeof(int));
    if(ogrNo == NULL || offset == NULL){
        printf("Memory not allocated.\n");
        exit(0);
    }
    if((file = fopen("index.txt", "r")) == NULL)
        printf("\nError! index.txt DOSYASI ACILAMADI. OLUSTURMAK ICIN 9\n");

    printf("\n======================================\n");
    printf("| Ogrenci No\tDers Kodu\tNotu |");
    printf("\n--------------------------------------\n");

    for(int i = 0; i < count; i++)
    {
        fscanf(file, "%d %d", &ogrNo[i], &offset[i]);
    }
    fclose(file);

    if((file2 = fopen("kayitlar.bin", "rb")) == NULL)
    {
        printf("\nError! kayitlar.bin DOSYASI ACILAMADI\n");
        exit(1);
    }

    int low=0, high=count-1, mid=(low+high)/2;
    while(low <= high)
    {
        if(ogrNo[mid]<search)
            low = mid+1;
        else if(ogrNo[mid]==search)
        {
            //Binary search ile bulunan ilk veri
            fseek(file2, offset[mid], SEEK_SET);
            fread(&s1, sizeof(ogrenci), 1, file2);
            printf("| %*d %*d %*d  |\n", 7, s1.ogrNo, 12, s1.dersKodu, 12, s1.puan);
            //Eger ayni ogrNo birden fazla varsa asagida yazilacaktir
            for(int i=0; i<=high; i++)
            {
                if(search == ogrNo[i] && mid != i)
                {
                    fseek(file2, offset[i], SEEK_SET);
                    fread(&s1, sizeof(ogrenci), 1, file2);
                    printf("| %*d %*d %*d  |\n", 7, s1.ogrNo, 12, s1.dersKodu, 12, s1.puan);
                }
            }
            break;
        }
        else
        high = mid-1;
        mid=(low+high)/2;
    }
    if(low>high)
    {
        printf("|\t\t\t\t     |\n");
        printf("|%*d NOLU OGRENCI BULUNAMADI!   |\n", 8, search);
        printf("|\t\t\t\t     |\n");
    }
    printf("======================================\n");
    fclose(file2);
    free(ogrNo);
    free(offset);
}

void kayitSil()
{
    ogrenci s1, *s;
    int search, j=0;
    int *ogrNo, *offset;
    FILE *file, *file2;

    printf("SILMEK ICIN OGRENCI NO YAZINIZ : ");
    scanf("%d", &search);

    if((file = fopen("index.txt", "r")) == NULL)
        printf("\nError! index.txt BULUNAMADI. Olusturmak icin 9");

    s = (ogrenci*)calloc(count-1, sizeof(ogrenci));
    ogrNo = (int*)calloc(count, sizeof(int));
    offset = (int*)calloc(count, sizeof(int));

    if(s == NULL || ogrNo == NULL || offset == NULL){
        printf("Memory not allocated.\n");
        exit(0);
    }
    for(int i = 0; i < count; i++)
        {
            fscanf(file, "%d %d", &ogrNo[i], &offset[i]);
        }
    fclose(file);

    if((file2 = fopen("kayitlar.bin", "rb")) == NULL)
        printf("Error! kayitlar.bin ACILAMADI");

    int low=0, high=count-1, mid=(low+high)/2;
    while(low<=high)
    {
        if(ogrNo[mid]<search)
            low = mid+1;
        else if(ogrNo[mid]==search)
        {
            for(int i=mid; i>0; i--) //for bottom
            {
                fseek(file2, offset[mid-i], SEEK_CUR);
                fread(&s1, sizeof(ogrenci), 1, file2);
                s[j] = s1;
                s[j].offset = offset[mid-i];
                j++;
                rewind(file2);
            }
            for(int i=mid; i<count-1; i++) //for up
            {
                fseek(file2, offset[i+1], SEEK_CUR);
                fread(&s1, sizeof(ogrenci), 1, file2);
                s[j] = s1;
                s[j].offset = offset[i+1];
                j++;
                rewind(file2);
            }
            fclose(file2);
            for(int i=0; i<count-1; i++)
            {
                for(j=i+1; j<count-1; j++)
                {
                    if(s[i].offset > s[j].offset)
                    {
                        s1 = s[i];
                        s[i] = s[j];
                        s[j] = s1;
                    }
                }
            }
            file2 = fopen("kayitlar.bin", "wb");
            for(int i=0; i<count-1; i++)
            {
                fwrite(&s[i], sizeof(ogrenci), 1, file2);
            }
            fclose(file2);
            printf("\n%d numarali ogrenci basariyla silindi!\n", search);
            break;
        }
        else
        high = mid-1;
        mid=(low+high)/2;
    }
    if(low>high)
    {
        printf("\n======================================");
        printf("\n|\t\t\t\t     |\n");
        printf("|%*d NOLU OGRENCI BULUNAMADI!   |\n", 8, search);
        printf("|\t\t\t\t     |\n");
        printf("======================================\n");
    }

    free(s);
    free(ogrNo);
    free(offset);
    counter();
    indexDosyasiOlustur();
}

void kayitGuncelle()
{
    ogrenci s1, s;
    int *ogrNo, *offset, search;
    FILE *file, *file2;

    printf("PUAN DEGISTIRMEK ICIN OGRENCI NO YAZINIZ : ");
    scanf("%d", &search);

    ogrNo = (int*)calloc(count, sizeof(int));
    offset = (int*)calloc(count, sizeof(int));
    if(ogrNo == NULL || offset == NULL){
        printf("Memory not allocated.\n");
        exit(0);
    }
    if((file = fopen("index.txt", "r")) == NULL)
        printf("\nError! index.txt BULUNAMADI. Olusturmak icin 9");

    for(int i = 0; i < count; i++)
        {
            fscanf(file, "%d %d", &ogrNo[i], &offset[i]);
        }
    fclose(file);

    if((file2 = fopen("kayitlar.bin", "rb+")) == NULL)
    {
        printf("Error! kayitlar.bin ACILAMADI");
        exit(1);
    }

    int low=0, high=count-1, mid=(low+high)/2, ans = 0;
    while(low<=high)
    {
        if(ogrNo[mid]<search)
            low = mid+1;
        else if(ogrNo[mid]==search)
        {
            //Binary search ile ilk bulunan veri
            fseek(file2, offset[mid], SEEK_SET);
            fread(&s1, sizeof(ogrenci), 1, file2);
            s = s1;
            printf("%d nolu dersinin puani guncelle? (1 Evet/0 Hayir) : ", s.dersKodu);
            scanf("%d", &ans);
            if(ans==1)
            {
                printf("Puani giriniz: ");
                scanf("%d", &s.puan);
                printf("\n");
                fseek(file2, offset[mid], SEEK_SET);
                fwrite(&s, sizeof(ogrenci), 1, file2);
            }
            for(int i=0; i<=high; i++) //Eger ayni ogrNo birden fazla varsa asagida degisecektir
            {
                if(search == ogrNo[i] && mid != i)
                {
                    fseek(file2, offset[i], SEEK_SET);
                    fread(&s1, sizeof(ogrenci), 1, file2);
                    s = s1;
                    printf("%d nolu dersinin puani guncelle? (1 Evet/0 Hayir) : ", s.dersKodu);
                    scanf("%d", &ans);
                    if(ans==1)
                    {
                        printf("Puani giriniz: ");
                        scanf("%d", &s.puan);
                        printf("\n");
                        fseek(file2, offset[i], SEEK_SET);
                        fwrite(&s, sizeof(ogrenci), 1, file2);
                    }
                }
            }
            break;
        }
        else
        high = mid-1;
        mid=(low+high)/2;
    }
    if(low>high)
    {
        printf("\n======================================");
        printf("\n|\t\t\t\t     |\n");
        printf("|%*d NOLU OGRENCI BULUNAMADI!   |\n", 8, search);
        printf("|\t\t\t\t     |\n");
        printf("======================================\n");
    }
    fclose(file2);
    free(ogrNo);
    free(offset);
}

void veriDosyasiniGoster()
{
    ogrenci s1;
    FILE *file;

    if((file = fopen("kayitlar.bin", "rb")) == NULL)
    {
        printf("Error! DOSYA ACILAMADI");
        exit(1);
    }

    printf("\n--------------------------------------\n");
    printf("| Ogrenci No\tDers Kodu\tNotu |");
    printf("\n--------------------------------------\n");

    while(fread(&s1, sizeof(ogrenci), 1, file)){
        printf("| %*d %*d %*d  |\n", 7, s1.ogrNo, 12, s1.dersKodu, 12, s1.puan);
    }
    printf("--------------------------------------\n");
    printf("\n");
    fclose(file);
}

void indeksDosyasiniGoster()
{
    int ogrNo, offset;
    FILE *file;
    file = fopen("index.txt", "r");
    if(file==NULL)
        printf("\n  DOSYA BULUNAMADI! index.txt olusturmak icin 9\n");
    else
    {
        printf("\n------------------------\n");
        printf("| Ogrenci No\tAdres  |");
        printf("\n------------------------\n");
        for(int i = 0; i < count; i++)
        {
            fscanf(file, "%d %d", &ogrNo, &offset);
            printf("| %*d%*d   |\n", 7, ogrNo, 11, offset);
        }
        printf("------------------------\n");
    }
    fclose(file);
}

void indeksDosyasiniSil()
{
    int del = remove("index.txt");
    if (!del)
    {
        printf("\n======================================");
        printf("\n|\t\t\t\t     |\n");
        printf("|  index.txt BASARIYLA SILINMISTIR!  |\n");
        printf("|\t\t\t\t     |\n");
        printf("======================================\n");
    }
    else
        printf("\n  DOSYA BULUNAMADI/SILINEMEDI!\n");
}

int main()
{
    int sec;
    printf("| Ogrenci Veri Sistemi v1.7 by massyakur & robtab (GitHub) |\n");
    printf("------------------------------------------------------------\n");
    counter();
    do{
        printf("\n-------------------------\n");
        printf("1. KAYIT OLUSTUR\n");
        printf("2. KAYIT EKLE\n");
        printf("3. KAYIT BUL\n");
        printf("4. KAYIT SIL\n");
        printf("5. KAYIT GUNCELLE\n");
        printf("6. VERI DOSYASI GOSTER\n");
        printf("7. INDEX DOSYASI GOSTER\n");
        printf("8. INDEX DOSYASI SIL\n");
        printf("9. INDEX DOSYASI OLUSTUR\n");
        printf("0. CIK\n");
        printf("-------------------------\n\n");

        printf("Istediginiz numarayi giriniz : ");
        scanf("%d", &sec);

        switch(sec){
            case 1:
                kayitOlustur();
            break;
            case 2:
                kayitEkle();
            break;
            case 3:
                kayitBul();
            break;
            case 4:
                kayitSil();
            break;
            case 5:
                kayitGuncelle();
            break;
            case 6:
                veriDosyasiniGoster();
            break;
            case 7:
                indeksDosyasiniGoster();
            break;
            case 8:
                indeksDosyasiniSil();
            break;
            case 9:
                indexDosyasiOlustur();
            break;
            default:
            break;
        }
    }while(sec!=0);
    return 0;
}
