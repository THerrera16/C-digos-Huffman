#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_SIZE 256
#define MAX_TEXT 10000

// Velázquez Herrera Maria Thais Itzel - 3CV4

typedef struct Node {
    char ch;
    int freq;
    struct Node *left, *right;
} Node;

typedef struct MinHeap {
    int size;
    Node **array;
} MinHeap;

typedef struct Code {
    char ch;
    char code[100];
} Code;

Node* newNode(char ch, int freq) {
    Node* node = (Node*)malloc(sizeof(Node));
    node->ch = ch;
    node->freq = freq;
    node->left = node->right = NULL;
    return node;
}

MinHeap* createMinHeap(int capacity) {
    MinHeap* minHeap = (MinHeap*)malloc(sizeof(MinHeap));
    minHeap->size = 0;
    minHeap->array = (Node**)malloc(capacity * sizeof(Node*));
    return minHeap;
}

void swapNode(Node** a, Node** b) {
    Node* t = *a;
    *a = *b;
    *b = t;
}

void minHeapify(MinHeap* minHeap, int idx) {
    int smallest = idx;
    int left = 2 * idx + 1;
    int right = 2 * idx + 2;
    int i;
    if (left < minHeap->size && minHeap->array[left]->freq < minHeap->array[smallest]->freq)
        smallest = left;
    if (right < minHeap->size && minHeap->array[right]->freq < minHeap->array[smallest]->freq)
        smallest = right;
    if (smallest != idx) {
        swapNode(&minHeap->array[smallest], &minHeap->array[idx]);
        minHeapify(minHeap, smallest);
    }
}

void insertMinHeap(MinHeap* minHeap, Node* node) {
    minHeap->size++;
    int i = minHeap->size - 1;
    while (i && node->freq < minHeap->array[(i - 1) / 2]->freq) {
        minHeap->array[i] = minHeap->array[(i - 1) / 2];
        i = (i - 1) / 2;
    }
    minHeap->array[i] = node;
}

Node* extractMin(MinHeap* minHeap) {
    Node* temp = minHeap->array[0];
    minHeap->array[0] = minHeap->array[minHeap->size - 1];
    minHeap->size--;
    minHeapify(minHeap, 0);
    return temp;
}

MinHeap* buildMinHeap(int freq[], int size) {
    MinHeap* minHeap = createMinHeap(size);
    int i;
    for (i = 0; i < size; i++)
        if (freq[i] > 0)
            insertMinHeap(minHeap, newNode((char)i, freq[i]));
    return minHeap;
}

Node* buildHuffmanTree(int freq[], int size) {
    Node *left, *right, *top;
    MinHeap* minHeap = buildMinHeap(freq, size);
    while (minHeap->size > 1) {
        left = extractMin(minHeap);
        right = extractMin(minHeap);
        top = newNode('$', left->freq + right->freq);
        top->left = left;
        top->right = right;
        insertMinHeap(minHeap, top);
    }
    return extractMin(minHeap);
}

void generateCodes(Node* root, char code[], int top, Code codes[], int *codeIndex) {
    int i;
    if (root->left) {
        code[top] = '0';
        generateCodes(root->left, code, top + 1, codes, codeIndex);
    }
    if (root->right) {
        code[top] = '1';
        generateCodes(root->right, code, top + 1, codes, codeIndex);
    }
    if (!root->left && !root->right) {
        codes[*codeIndex].ch = root->ch;
        code[top] = '\0';
        strcpy(codes[*codeIndex].code, code);
        (*codeIndex)++;
    }
}

void saveEncodedFile(char *text, Code codes[], int codeCount, int originalBits) {
    FILE *fp = fopen("codificado.txt", "w");
    if (!fp) {
        printf("Error opening codificado.txt\n");
        return;
    }
    int encodedBits = 0;
    int i, j;
    for (i = 0; text[i]; i++)
        for (j = 0; j < codeCount; j++)
            if (text[i] == codes[j].ch) {
                fprintf(fp, "%s", codes[j].code);
                encodedBits += strlen(codes[j].code);
            }
    printf("\nOriginal size: %d bits\n", originalBits);
    printf("Compressed size: %d bits\n", encodedBits);
    printf("Compression ratio: %.2f%%\n", (float)encodedBits / originalBits * 100);
    fclose(fp);
}

void decodeFile(Node* root, char *encodedText) {
    FILE *fp = fopen("decodificado.txt", "w");
    if (!fp) {
        printf("Error opening decodificado.txt\n");
        return;
    }
    Node* curr = root;
    int i;
    for (i = 0; encodedText[i]; i++) {
        curr = (encodedText[i] == '0') ? curr->left : curr->right;
        if (!curr->left && !curr->right) {
            fprintf(fp, "%c", curr->ch);
            curr = root;
        }
    }
    fclose(fp);
}

int main() {
    FILE *fp = fopen("input.txt", "r");
    if (!fp) {
        printf("Error opening input.txt\n");
        return 1;
    }
    char text[MAX_TEXT];
    int freq[MAX_SIZE] = {0};
    int c, textLength = 0;
    int i;
    while ((c = fgetc(fp)) != EOF && textLength < MAX_TEXT - 1) {
        text[textLength++] = c;
        freq[c]++;
    }
    text[textLength] = '\0';
    fclose(fp);
    Node* root = buildHuffmanTree(freq, MAX_SIZE);
    Code codes[MAX_SIZE];
    int codeIndex = 0;
    char code[100];
    generateCodes(root, code, 0, codes, &codeIndex);
    printf("Character frequencies:\n");
    for (i = 0; i < MAX_SIZE; i++)
        if (freq[i] > 0)
            printf("'%c': %d\n", (char)i, freq[i]);
    printf("\nHuffman Codes:\n");
    for (i = 0; i < codeIndex; i++)
        printf("'%c': %s\n", codes[i].ch, codes[i].code);
    saveEncodedFile(text, codes, codeIndex, textLength * 8);
    fp = fopen("codificado.txt", "r");
    if (!fp) {
        printf("Error opening codificado.txt\n");
        return 1;
    }
    char encodedText[MAX_TEXT];
    textLength = 0;
    while ((c = fgetc(fp)) != EOF && textLength < MAX_TEXT - 1)
        encodedText[textLength++] = c;
    encodedText[textLength] = '\0';
    fclose(fp);
    decodeFile(root, encodedText);
    printf("\nDecoding complete. Check decodificado.txt\n");
    return 0;
}
