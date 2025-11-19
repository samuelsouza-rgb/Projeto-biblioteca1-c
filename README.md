# Projeto-biblioteca1-c
/*
  biblioteca.c
  Projeto Prático - Laboratório de Programação Estruturada
  Tema: Biblioteca Pessoal (cadastro de livros)
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ARQUIVO_BIB "biblioteca.dat"
#define MAX_STR 100

typedef struct {
    char titulo[MAX_STR];
    char autor[MAX_STR];
    int isbn;
    float preco;
} Livro;

void limpaBuffer(void);
long tamanho(FILE *fp);
void cadastrar(FILE *fp);
void consultar(FILE *fp);
void listar_todos(FILE *fp);
void ler_string(const char *prompt, char *dest, size_t size);

void limpaBuffer(void) {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) { }
}

long tamanho(FILE *fp) {
    if (fp == NULL) return 0;
    long cur = ftell(fp);
    fseek(fp, 0, SEEK_END);
    long end = ftell(fp);
    fseek(fp, cur, SEEK_SET);
    return end / sizeof(Livro);
}

void ler_string(const char *prompt, char *dest, size_t size) {
    printf("%s", prompt);
    if (fgets(dest, (int)size, stdin) != NULL) {
        dest[strcspn(dest, "\n")] = '\0';
    } else {
        dest[0] = '\0';
        limpaBuffer();
    }
}

void cadastrar(FILE *fp) {
    if (fp == NULL) {
        printf("Arquivo não aberto.\n");
        return;
    }

    Livro L;
    ler_string("Título: ", L.titulo, sizeof(L.titulo));
    ler_string("Autor:  ", L.autor, sizeof(L.autor));

    printf("ISBN (numeros): ");
    if (scanf("%d", &L.isbn) != 1) {
        printf("ISBN inválido.\n");
        limpaBuffer();
        return;
    }
    limpaBuffer();

    printf("Preço: ");
    if (scanf("%f", &L.preco) != 1) {
        printf("Preço inválido.\n");
        limpaBuffer();
        return;
    }
    limpaBuffer();

    fseek(fp, 0, SEEK_END);
    fwrite(&L, sizeof(Livro), 1, fp);
    printf("Livro cadastrado com sucesso.\n");
}

void consultar(FILE *fp) {
    long total = tamanho(fp);
    if (total == 0) {
        printf("Nenhum registro.\n");
        return;
    }

    long idx;
    printf("Índice (1 a %ld): ", total);
    if (scanf("%ld", &idx) != 1) {
        printf("Entrada inválida.\n");
        limpaBuffer();
        return;
    }
    limpaBuffer();

    if (idx < 1 || idx > total) {
        printf("Índice inválido.\n");
        return;
    }

    Livro L;
    fseek(fp, (idx - 1) * sizeof(Livro), SEEK_SET);
    fread(&L, sizeof(Livro), 1, fp);

    printf("\n---- Registro %ld ----\n", idx);
    printf("Título: %s\n", L.titulo);
    printf("Autor : %s\n", L.autor);
    printf("ISBN  : %d\n", L.isbn);
    printf("Preço : R$ %.2f\n\n", L.preco);
}

void listar_todos(FILE *fp) {
    long total = tamanho(fp);
    if (total == 0) {
        printf("Nenhum registro.\n");
        return;
    }

    Livro L;
    fseek(fp, 0, SEEK_SET);

    for (long i = 0; i < total; i++) {
        fread(&L, sizeof(Livro), 1, fp);
        printf("[%ld] %s | %s | ISBN: %d | R$ %.2f\n",
               i + 1, L.titulo, L.autor, L.isbn, L.preco);
    }
}

int main(void) {
    FILE *fp = fopen(ARQUIVO_BIB, "r+b");
    if (!fp) fp = fopen(ARQUIVO_BIB, "w+b");

    int opc;
    do {
        printf("\n==== Biblioteca ====\n");
        printf("1 - Cadastrar\n");
        printf("2 - Consultar por índice\n");
        printf("3 - Quantidade\n");
        printf("4 - Listar todos\n");
        printf("0 - Sair\n");
        printf("Opção: ");

        if (scanf("%d", &opc) != 1) {
            printf("Entrada inválida.\n");
            limpaBuffer();
            continue;
        }
        limpaBuffer();

        switch (opc) {
            case 1: cadastrar(fp); break;
            case 2: consultar(fp); break;
            case 3: printf("Total: %ld\n", tamanho(fp)); break;
            case 4: listar_todos(fp); break;
            case 0: break;
            default: printf("Opção inválida.\n");
        }
    } while (opc != 0);

    fclose(fp);
    return 0;
}
