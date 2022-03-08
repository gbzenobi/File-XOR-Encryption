Greetings, today I come with a small library (which I hope to continue and improve in many aspects), which serves mainly to encrypt small resources and in the future hopefully they will be of more importance.

What use is this to me?
Well I will begin by saying that I was programming a minigame in SDL with C++ and I wanted to encrypt the game resources (like images and other files) because when a game is distributed we are not interested in the player to see all these resources...
I came up with the idea of doing it on my own, this is my method and as I always say, you can improve it(in fact that's the idea) in the end this is more a "proof of concept", I hope soon to document more the use of SDL in some aspects... since I didn't find too much about how to load resources except RWops.

Another interesting thing is that it encrypts without overwriting, with this I mean that if we have 2 files one with name A.ext and B.ext when encrypting it will be like this: A(0).ext and B(1).ext, the same when decrypting...

Encrypted:

![e1](/1.png)

In the same way you can decrypt it (with SDL I managed to make it runtime, so the user would not notice it
<br/>
Decrypted:

![e2](/2.png)

Code:
```C
/*******************************************
*
*
* Author : NvK
* lib : FileEncryptionOps
* Version : 0.9
* Description : Trabaja con imágenes principalmente, permitiendo encriptar/desencriptar,
* Compatible with SDL to work with RWops and resources.
* Date : 24-2-2014
*
********************************************/
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <fcntl.h>

#ifndef __FILE_ENCRYPTION_OPS__
	#define __FILE_ENCRYPTION_OPS__

	#define BEGIN_FILE_POS          (0x0001)
	#define WRITE_TEXT              ("w")
	#define WRITE_BINARY            ("wb")
	#define READ_TEXT               ("r")
	#define READ_BINARY             ("rb")

	long GetFileSize(const char *filePath);
	char *strlast(char *str, int last_n);
	void WriteXORFileEx(const char *InputFile, const char *OutputFile);
	void XOR_WriteFile_Encryption(const char *FilePath);

    #define __XOR_KEY__
    #ifdef __XOR_KEY__
        const char xor_key[]= {'N', 'V', 'K'};
	#endif
#endif

long GetFileSize(const char *filePath)
{
	int fp= open(filePath, O_RDONLY);
	return (lseek(fp, 0x0000, SEEK_END));
}

void strlast(char *str, int last_n, char output_buf[])
{
    volatile register int __ptr_length __asm__("ebx");
    register int __ptr_last_n __asm__("ecx");

    __asm__(
            "mov %%eax, (%%esp);"
            "call _strlen;"
            "movl %%eax, %%ebx;"
            ::"a"(str)
        );
    __asm__("mov %%edx, %%ecx;"
            "sub %%ebx, %%ecx;"
            "neg %%ecx;" :: "d"(last_n) );

    for(int i=0; i<=last_n; i++)
        __asm__ volatile(";":"=r"(output_buf[i]):"a"(str[ __ptr_last_n++ ]));
}

void WriteXORFileEx(const char *InputFile, const char *OutputFile)
{
    int fileSize= GetFileSize(InputFile);
    FILE *fInput= fopen(InputFile, READ_BINARY); // abre el archivo en modo binario
    FILE *fOutput= fopen(OutputFile, WRITE_BINARY); // escribe el archivo en modo binario
    int Buff[fileSize];
    char EncBuf[fileSize];

    char *Write= (char *)malloc(fileSize);
    fread(Write, fileSize, BEGIN_FILE_POS, fInput);

    for(int b=0;b<fileSize;b++)
    {
        EncBuf[b]= (Write[b]^xor_key[b % (sizeof(xor_key))]);
    }
    fwrite(EncBuf, BEGIN_FILE_POS, fileSize, fOutput);

    fclose(fInput);
    fclose(fOutput);
}

void XOR_WriteFile_Encryption(const char *FilePath)
{
    char filesPath[460];
    char fileName[360];
    char fileExt[4];
    char *fileNum;
    char filesToEncrypt[360];
    char filesEncrypted[360];

    WIN32_FIND_DATA FindFileData;
    HANDLE hFile;

    sprintf(filesPath, FilePath);
    chdir(FilePath);
    hFile= FindFirstFile("*.*", &FindFileData);

    volatile UINT nFiles __asm__("eax") = 0; // así se preserva

    if(hFile!=INVALID_HANDLE_VALUE)
    {
        do{
            if(strcmp(FindFileData.cFileName, ".")&&
               (strcmp("..", FindFileData.cFileName)) )
            {
                sprintf(filesToEncrypt, "%s%s", filesPath, FindFileData.cFileName);
                snprintf(fileName, strlen(FindFileData.cFileName)-0x3, "%s", FindFileData.cFileName); // quitar la extención del archivo
                strlast(FindFileData.cFileName, 4, fileExt);

                itoa(nFiles, fileNum, 0x4);
                sprintf(filesEncrypted, "%s%s(%s)%s",
                        filesPath,                      // Copia la ruta del archivo
                        fileName,                       // Copia el nombre
                        fileNum,                        // Copia el numero
                        fileExt                         // agrega la extención
                        );

                printf("%s\n", filesToEncrypt);

                WriteXORFileEx(filesToEncrypt, filesEncrypted);
                __asm__ volatile(
                         "inc %%eax;" : "=r"(nFiles) : "a"(nFiles)
                        );

            }
        } while(FindNextFile(hFile, &FindFileData));
    }
    free(fileNum);

    FindClose(hFile);
}
```
Example
```C
#include "FileEncryptionOps.h"
int main()
{
	XOR_WriteFile_Encryption("C:\\Objs\\"); // It will encrypt everything in the "Objs" folder on disk C:
	return 0;
}
```

***

My original post: https://indetectables.net/viewtopic.php?t=49255
<br/>
Date: 24-2-2014
