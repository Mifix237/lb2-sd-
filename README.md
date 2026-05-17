# Лабораторная работа №1: Структура данных
## Задание:
<img width="1212" height="283" alt="image" src="https://github.com/user-attachments/assets/cf7f048e-9a81-4674-8189-77e987270b0e" />
## Листинг программы: 

``` C++
#include <iostream>
#include <ctime>
#include <chrono>

extern "C" {
#include <mkl.h>
}

using namespace std;

const int N = 2048;
const int BLOCK = 64;

// Заполнение матриц
void fillMatrix(double* a, double* b, double* c) {

    for (int i = 0; i < N; i++) {

        for (int j = 0; j < N; j++) {

            a[i * N + j] = rand() % 5;
            b[i * N + j] = rand() % 5;

            c[i * N + j] = 0;
        }
    }
}

// 1 вариант — обычное умножение
void simpleMultiply(double* a, double* b, double* c) {

    for (int i = 0; i < N; i++) {

        for (int j = 0; j < N; j++) {

            for (int k = 0; k < N; k++) {

                c[i * N + j] +=
                    a[i * N + k] *
                    b[k * N + j];
            }
        }
    }
}

// 2 вариант — BLAS
void blasMultiply(double* a, double* b, double* c) {

    cblas_dgemm(CblasRowMajor,CblasNoTrans,CblasNoTrans,N,N,N,1.0,a,N,b,N,0.0,c,N);
}

// 3 вариант — блочное умножение
void blockMultiply(double* a, double* b, double* c) {

    for (int ii = 0; ii < N; ii += BLOCK) {

        for (int jj = 0; jj < N; jj += BLOCK) {

            for (int kk = 0; kk < N; kk += BLOCK) {

                for (int i = ii; i < min(ii + BLOCK, N); i++) {

                    for (int k = kk; k < min(kk + BLOCK, N); k++) {

                        double temp = a[i * N + k];

                        for (int j = jj; j < min(jj + BLOCK, N); j++) {

                            c[i * N + j] +=
                                temp * b[k * N + j];
                        }
                    }
                }
            }
        }
    }
}

// Подсчёт времени и MFLOPS
void testAlgorithm(
    void (*multiply)(double*, double*, double*),
    double* a,
    double* b,
    double* c,
    string name
) {

    // Обнуление матрицы результата
    for (int i = 0; i < N * N; i++) {
        c[i] = 0;
    }

    auto start = chrono::high_resolution_clock::now();

    multiply(a, b, c);

    auto end = chrono::high_resolution_clock::now();

    double time =
        chrono::duration<double>(end - start).count();

    double operations = 2.0 * N * N * N;

    double mflops = operations / time / 1e6;

    cout << "\n" << name << endl;
    cout << "Time: " << time << " sec" << endl;
    cout << "MFLOPS: " << mflops << endl;
}

int main() {

    setlocale(LC_ALL, "");
    srand(time(NULL));

    double* a = new double[N * N];
    double* b = new double[N * N];
    double* c = new double[N * N];

    fillMatrix(a, b, c);

    testAlgorithm(simpleMultiply, a, b, c,
        "Option 1 is the usual multiplication");

    testAlgorithm(blasMultiply, a, b, c,
        "Option 2: BLAS");

    testAlgorithm(blockMultiply, a, b, c,
        "3 Option: Block Multiplication");

    delete[] a;
    delete[] b;
    delete[] c;

    return 0;
}
```

## Результат выполнения программы:
<img width="578" height="262" alt="image" src="https://github.com/user-attachments/assets/b36f9214-2bd5-4b78-9c0b-732eb08d5f0a" />

