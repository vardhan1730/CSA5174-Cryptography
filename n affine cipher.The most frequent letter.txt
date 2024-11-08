#include <stdio.h>
#include <string.h>
#include <ctype.h>

// Function to find the greatest common divisor (gcd)
int gcd(int a, int b) {
    while (b != 0) {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}

// Function to find the modular inverse of a number modulo 26
int modInverse(int a, int m) {
    for (int x = 1; x < m; x++) {
        if ((a * x) % m == 1) {
            return x;
        }
    }
    return -1; // No modular inverse exists if this returns -1
}

// Function to decrypt the ciphertext using the affine cipher
void affineDecrypt(char *ciphertext, int a, int b, char *plaintext) {
    int i;
    int a_inv = modInverse(a, 26); // Find the modular inverse of 'a' mod 26

    if (a_inv == -1) {
        printf("Error: No modular inverse for 'a'. Decryption impossible.\n");
        return;
    }

    for (i = 0; ciphertext[i] != '\0'; i++) {
        char c = ciphertext[i];

        if (isalpha(c)) {
            if (isupper(c)) {
                plaintext[i] = (a_inv * (c - 'A' - b + 26)) % 26 + 'A';
            } else {
                plaintext[i] = (a_inv * (c - 'a' - b + 26)) % 26 + 'a';
            }
        } else {
            plaintext[i] = c; // Non-alphabet characters remain unchanged
        }
    }
    plaintext[i] = '\0';  // Null-terminate the plaintext
}

// Function to solve the system of equations and determine 'a' and 'b'
void breakAffineCipher() {
    int a, b;
    
    // Ciphertext mapping: B -> E (index 1 -> 4), U -> T (index 20 -> 19)
    int C_B = 1, C_U = 20;    // Ciphertext letters B and U
    int p_B = 4, p_U = 19;    // Corresponding plaintext letters E and T

    // Try all values of 'a' and 'b' that satisfy the affine cipher equations
    for (a = 0; a < 26; a++) {
        if (gcd(a, 26) == 1) { // 'a' must be coprime with 26
            for (b = 0; b < 26; b++) {
                // Check the first equation: (a * p_B + b) % 26 == C_B
                if ((a * p_B + b) % 26 == C_B) {
                    // Check the second equation: (a * p_U + b) % 26 == C_U
                    if ((a * p_U + b) % 26 == C_U) {
                        printf("Found key values: a = %d, b = %d\n", a, b);
                        return;
                    }
                }
            }
        }
    }

    printf("No valid keys found. Decryption not possible.\n");
}

int main() {
    char ciphertext[256], plaintext[256];
    int a, b;

    // Break the affine cipher based on the frequency analysis of B -> E, U -> T
    breakAffineCipher();

    // Ask the user for the ciphertext and keys a, b
    printf("Enter the ciphertext: ");
    fgets(ciphertext, sizeof(ciphertext), stdin);
    ciphertext[strcspn(ciphertext, "\n")] = '\0'; // Remove newline from input

    // Input the keys 'a' and 'b' after the system of equations is solved
    printf("Enter the value for 'a': ");
    scanf("%d", &a);

    printf("Enter the value for 'b': ");
    scanf("%d", &b);

    // Decrypt the ciphertext using the affine cipher with the found keys
    affineDecrypt(ciphertext, a, b, plaintext);

    // Display the resulting plaintext
    printf("Decrypted text: %s\n", plaintext);

    return 0;
}
