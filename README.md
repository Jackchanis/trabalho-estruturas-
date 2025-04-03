/* processo.h */
#ifndef PROCESSO_H
#define PROCESSO_H

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_PROCESSOS 1000

// Estrutura do Processo
typedef struct {
    int id;
    char numero[21];
    char data_ajuizamento[20];
    int id_classe;
    int id_assunto;
    int ano_eleicao;
} Processo;

// Funções do TAD
void lerArquivoCSV(const char *nomeArquivo, Processo processos[], int *quantidade);
void ordenarPorId(Processo processos[], int quantidade);
void ordenarPorData(Processo processos[], int quantidade);
int contarPorClasse(Processo processos[], int quantidade, int id_classe);
int contarAssuntos(Processo processos[], int quantidade);
void listarMultiplosAssuntos(Processo processos[], int quantidade);
int calcularDiasTramitacao(const char *data);
void salvarCSV(const char *nomeArquivo, Processo processos[], int quantidade);

#endif

/* processo.c */
#include "processo.h"

void lerArquivoCSV(const char *nomeArquivo, Processo processos[], int *quantidade) {
    FILE *arquivo = fopen(nomeArquivo, "r");
    if (!arquivo) {
        printf("Erro ao abrir o arquivo!\n");
        exit(1);
    }

    char linha[200];
    fgets(linha, sizeof(linha), arquivo);
    *quantidade = 0;

    while (fgets(linha, sizeof(linha), arquivo) && *quantidade < MAX_PROCESSOS) {
        Processo p;
        sscanf(linha, "%d,%20[^,],%19[^,],%d,%d,%d", &p.id, p.numero, p.data_ajuizamento, &p.id_classe, &p.id_assunto, &p.ano_eleicao);
        processos[(*quantidade)++] = p;
    }
    fclose(arquivo);
}

void ordenarPorId(Processo processos[], int quantidade) {
    for (int i = 0; i < quantidade - 1; i++) {
        for (int j = i + 1; j < quantidade; j++) {
            if (processos[i].id > processos[j].id) {
                Processo temp = processos[i];
                processos[i] = processos[j];
                processos[j] = temp;
            }
        }
    }
}

void ordenarPorData(Processo processos[], int quantidade) {
    for (int i = 0; i < quantidade - 1; i++) {
        for (int j = i + 1; j < quantidade; j++) {
            struct tm data1 = {0}, data2 = {0};
            sscanf(processos[i].data_ajuizamento, "%d-%d-%d", &data1.tm_year, &data1.tm_mon, &data1.tm_mday);
            sscanf(processos[j].data_ajuizamento, "%d-%d-%d", &data2.tm_year, &data2.tm_mon, &data2.tm_mday);
            data1.tm_year -= 1900; data1.tm_mon -= 1;
            data2.tm_year -= 1900; data2.tm_mon -= 1;
            
            if (difftime(mktime(&data1), mktime(&data2)) > 0) {
                Processo temp = processos[i];
                processos[i] = processos[j];
                processos[j] = temp;
            }
        }
    }
}

int contarPorClasse(Processo processos[], int quantidade, int id_classe) {
    int count = 0;
    for (int i = 0; i < quantidade; i++) {
        if (processos[i].id_classe == id_classe) {
            count++;
        }
    }
    return count;
}

int contarAssuntos(Processo processos[], int quantidade) {
    int assuntos[MAX_PROCESSOS] = {0};
    int count = 0;
    for (int i = 0; i < quantidade; i++) {
        if (!assuntos[processos[i].id_assunto]) {
            assuntos[processos[i].id_assunto] = 1;
            count++;
        }
    }
    return count;
}

void listarMultiplosAssuntos(Processo processos[], int quantidade) {
    for (int i = 0; i < quantidade; i++) {
        if (processos[i].id_assunto > 1) {
            printf("ID: %d - Número: %s - Assunto: %d\n", processos[i].id, processos[i].numero, processos[i].id_assunto);
        }
    }
}

int calcularDiasTramitacao(const char *data) {
    struct tm dataProcesso = {0};
    sscanf(data, "%d-%d-%d", &dataProcesso.tm_year, &dataProcesso.tm_mon, &dataProcesso.tm_mday);
    dataProcesso.tm_year -= 1900;
    dataProcesso.tm_mon -= 1;

    time_t agora = time(NULL);
    struct tm *dataAtual = localtime(&agora);

    time_t t1 = mktime(&dataProcesso);
    time_t t2 = mktime(dataAtual);

    return (t2 - t1) / (60 * 60 * 24);
}

void salvarCSV(const char *nomeArquivo, Processo processos[], int quantidade) {
    FILE *arquivo = fopen(nomeArquivo, "w");
    if (!arquivo) {
        printf("Erro ao abrir arquivo para escrita!\n");
        return;
    }
    fprintf(arquivo, "id,numero,data_ajuizamento,id_classe,id_assunto,ano_eleicao\n");
    for (int i = 0; i < quantidade; i++) {
        fprintf(arquivo, "%d,%s,%s,%d,%d,%d\n", processos[i].id, processos[i].numero, processos[i].data_ajuizamento, processos[i].id_classe, processos[i].id_assunto, processos[i].ano_eleicao);
    }
    fclose(arquivo);
}

/* main.c */
#include "processo.h"

int main() {
    Processo processos[MAX_PROCESSOS];
    int quantidade;
    
    lerArquivoCSV("processo.csv", processos, &quantidade);
    
    ordenarPorId(processos, quantidade);
    salvarCSV("ordenado_por_id.csv", processos, quantidade);
    
    ordenarPorData(processos, quantidade);
    salvarCSV("ordenado_por_data.csv", processos, quantidade);
    
    int id_classe = 12377;
    printf("Total de processos na classe %d: %d\n", id_classe, contarPorClasse(processos, quantidade, id_classe));
    
    printf("Total de assuntos distintos: %d\n", contarAssuntos(processos, quantidade));
    
    printf("Processos com múltiplos assuntos:\n");
    listarMultiplosAssuntos(processos, quantidade);
    
    printf("Dias em tramitação do primeiro processo: %d dias\n", calcularDiasTramitacao(processos[0].data_ajuizamento));
    
    return 0;
}
