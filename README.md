# SIMD instruction

## Opis

Program demonstruje różne operacje na danych oraz pokazuje, jak dane są przechowywane w pamięci w języku C++. Wykorzystuje strukturę `Data` i unię `Vector4` do ilustracji wyrównania pamięci oraz operacji na danych za pomocą instrukcji SIMD (Single Instruction, Multiple Data).

## Struktura Kodu

### Struktura `Data`

Struktura `Data` jest wyrównana do granicy 16 bajtów dzięki dyrektywie `alignas(16)`. Składa się z dwóch pól typu `char` i jednego pola typu `int`.

```cpp
struct alignas(16) Data // 16 -> 0 at the end of address in hex
{
    char c; // 1
    char a; // 1
    // +2 padding to align the next member to a 4-byte boundary
    int b;  // 4
};
```

### Unia `Vector4`

Unia `Vector4` zawiera cztery pola typu `float` oraz jedno pole typu `__m128`, które jest wyrównane do granicy 16 bajtów dzięki wewnętrznemu wyrównaniu typu `__m128`.

```cpp
union Vector4
{
    struct
    {
        float x;
        float y;
        float z;
        float w;
    };

    __m128 data;    // __m128 is aligned, so the Vector4 will be aligned by default
};
```

### Wykorzystanie SIMD

W funkcji `main` program ilustruje, jak wykorzystać instrukcje SIMD do wykonywania operacji na wektorach typu `float`.

```cpp
int main()
{
    void* data = malloc(64);    // alokacja 64 bajtów

    float a[4] = { 1.0f, 0.0f, 2.0f, 3.0f };
    float b[4] = { 2.0f, 3.0f, 4.0f, 5.0f };
    float c[4];

    // Operacje na tablicach
    c[0] = a[0] + b[0];
    c[1] = a[1] + b[1];
    c[2] = a[2] + b[2];
    c[3] = a[3] + b[3];

    // Operacje SIMD
    __m128 a_sse = _mm_load_ps(a);
    __m128 b_sse = _mm_load_ps(b);
    __m128 c_sse = _mm_add_ps(a_sse, b_sse);

    _mm_store_ps(c, c_sse);

    return 0;
}
```

### Operacje SIMD

Instrukcje SIMD umożliwiają wykonywanie operacji na wielu danych jednocześnie, co jest wydajne dla obliczeń wektorowych.

```cpp
__m128 a_sse = _mm_load_ps(a);  // Ładowanie danych do rejestru SIMD
__m128 b_sse = _mm_load_ps(b);
__m128 c_sse = _mm_add_ps(a_sse, b_sse);  // Dodawanie wektorów
_mm_store_ps(c, c_sse);  // Zapis wyników do tablicy
```

## Komentarze i Wyjaśnienia

### Wyrównanie i Układ Pamięci

Wyrównanie (`alignment`) to proces dostosowywania danych w pamięci do granic pamięci w celu zwiększenia wydajności dostępu. Struktura `Data` jest wyrównana do 16 bajtów, co zapewnia, że jej adres kończy się na 0 w reprezentacji szesnastkowej.

### Operacje Bitowe

Program zawiera przykłady operacji bitowych, które są powszechnie używane do manipulacji danymi na poziomie bitów.

```cpp
// Przesunięcie bitowe w prawo i w lewo
// 0011 0101 >> 3 = 0000 0110 << 5 = 1100 0000
```

### Przykłady Operacji na Danych

```cpp
// Przesunięcie i maskowanie danych
// qword = 8
// int64 data, data >> 3 * 8 << 4 * 8
// dword = 4 + byte = 1
// abbb + (b)
// qword = 8
// int64 data & 0xFF000000000000
```

### Ilustracje Operacji SIMD

Przykłady operacji na danych za pomocą SIMD:

```cpp
// Dodawanie wektorów za pomocą SIMD
a + b = c
c = _mm_add_ps(a, b);

// Mnożenie skalarne wektorów i sumowanie elementów
dot( v1, v2 ) = v1.x * v2.x + v1.y * v2.y + v1.z * v2.z + v1.w * v1.w
c = _mm_mul_ps(v1, v2);

// Mnożenie i sumowanie wielu wektorów
dot x4
c12 = _mm_mul_ps(v1, v2);
c34 = _mm_mul_ps(v3, v4);
c56 = _mm_mul_ps(v5, v6);
c78 = _mm_mul_ps(v7, v8);

// Przekształcenie danych z AoS do SoA (Array of Structures do Structure of Arrays)
c12[0] c34[0] c56[0] c78[0]
+      +      +      +
c12[1] c34[1] c56[1] c78[1]
+      +      +      +
c12[2] c34[2] c56[2] c78[2]
+      +      +      +
c12[3] c34[3] c56[3] c78[3]
=      =      =      =
c      c      c      c

xxxx
++++
yyyy
++++
zzzz
++++
wwww
====
cccc
```

## Podsumowanie

Program demonstruje różne aspekty operacji na danych oraz ich przechowywania w pamięci, z uwzględnieniem wyrównania pamięci oraz wykorzystania instrukcji SIMD dla optymalizacji obliczeń wektorowych. Komentarze w kodzie i wyjaśnienia pomagają zrozumieć, jak działają operacje na danych na poziomie bitowym oraz jak dane są rozmieszczone w pamięci.
