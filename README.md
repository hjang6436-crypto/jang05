#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define SIZE 1000

// ======================= AVL / BST 구조체 정의 ========================

typedef struct Node {
    int key;
    int height;
    struct Node* left;
    struct Node* right;
} Node;

// ========================= 유틸리티 함수 ==============================

int max(int a, int b) {
    return (a > b) ? a : b;
}

int height(Node* n) {
    return n ? n->height : 0;
}

Node* newNode(int key) {
    Node* node = (Node*)malloc(sizeof(Node));
    node->key = key;
    node->left = node->right = NULL;
    node->height = 1;
    return node;
}

// ==================== AVL 회전 및 균형 유지 ===========================

Node* rightRotate(Node* y) {
    Node* x = y->left;
    Node* T2 = x->right;

    x->right = y;
    y->left = T2;

    y->height = max(height(y->left), height(y->right)) + 1;
    x->height = max(height(x->left), height(x->right)) + 1;

    return x;
}

Node* leftRotate(Node* x) {
    Node* y = x->right;
    Node* T2 = y->left;

    y->left = x;
    x->right = T2;

    x->height = max(height(x->left), height(x->right)) + 1;
    y->height = max(height(y->left), height(y->right)) + 1;

    return y;
}

int getBalance(Node* n) {
    return n ? height(n->left) - height(n->right) : 0;
}

// =================== BST / AVL 삽입 구현 ===========================

Node* insertBST(Node* node, int key) {
    if (!node) return newNode(key);

    if (key < node->key)
        node->left = insertBST(node->left, key);
    else
        node->right = insertBST(node->right, key);

    return node;
}

Node* insertAVL(Node* node, int key) {
    if (!node) return newNode(key);

    if (key < node->key)
        node->left = insertAVL(node->left, key);
    else if (key > node->key)
        node->right = insertAVL(node->right, key);
    else
        return node;  // 중복 X

    node->height = 1 + max(height(node->left), height(node->right));

    int balance = getBalance(node);

    // 회전 조건
    if (balance > 1 && key < node->left->key)
        return rightRotate(node);
    if (balance < -1 && key > node->right->key)
        return leftRotate(node);
    if (balance > 1 && key > node->left->key) {
        node->left = leftRotate(node->left);
        return rightRotate(node);
    }
    if (balance < -1 && key < node->right->key) {
        node->right = rightRotate(node->right);
        return leftRotate(node);
    }

    return node;
}

// ======================== 탐색 함수 ==============================

int searchArray(int* arr, int size, int key, int* comparisons) {
    *comparisons = 0;
    for (int i = 0; i < size; i++) {
        (*comparisons)++;
        if (arr[i] == key)
            return 1;
    }
    return 0;
}

int searchTree(Node* root, int key, int* comparisons) {
    *comparisons = 0;
    while (root) {
        (*comparisons)++;
        if (key == root->key)
            return 1;
        else if (key < root->key)
            root = root->left;
        else
            root = root->right;
    }
    return 0;
}

// ===================== 데이터셋 생성 함수 ==========================

void generate_random_unique(int* arr, int size) {
    int used[10001] = {0};
    int count = 0;
    while (count < size) {
        int num = rand() % 10001;
        if (!used[num]) {
            arr[count++] = num;
            used[num] = 1;
        }
    }
}

void generate_sorted(int* arr) {
    for (int i = 0; i < SIZE; i++)
        arr[i] = i;
}

void generate_reverse_sorted(int* arr) {
    for (int i = 0; i < SIZE; i++)
        arr[i] = SIZE - 1 - i;
}

void generate_formula(int* arr) {
    for (int i = 0; i < SIZE; i++)
        arr[i] = i * (i % 2 + 2);
}

// ======================= 실험 실행 함수 ============================

void run_experiment(int* data, const char* title) {
    int search_keys[SIZE];
    for (int i = 0; i < SIZE; i++)
        search_keys[i] = rand() % 10001;

    // 배열 삽입
    int* array = (int*)malloc(sizeof(int) * SIZE);
    for (int i = 0; i < SIZE; i++)
        array[i] = data[i];

    // BST / AVL 삽입
    Node* bst_root = NULL;
    Node* avl_root = NULL;
    for (int i = 0; i < SIZE; i++) {
        bst_root = insertBST(bst_root, data[i]);
        avl_root = insertAVL(avl_root, data[i]);
    }

    long long total_cmp_array = 0;
    long long total_cmp_bst = 0;
    long long total_cmp_avl = 0;

    for (int i = 0; i < SIZE; i++) {
        int cmp;

        searchArray(array, SIZE, search_keys[i], &cmp);
        total_cmp_array += cmp;

        searchTree(bst_root, search_keys[i], &cmp);
        total_cmp_bst += cmp;

        searchTree(avl_root, search_keys[i], &cmp);
        total_cmp_avl += cmp;
    }

    printf("\n[%s] 평균 탐색 비교 횟수:\n", title);
    printf("  - 배열(선형탐색): %.2f\n", total_cmp_array / (float)SIZE);
    printf("  - BST: %.2f\n", total_cmp_bst / (float)SIZE);
    printf("  - AVL: %.2f\n", total_cmp_avl / (float)SIZE);

    free(array);
}

// ============================ main ================================

int main() {
    srand(time(NULL));

    int data1[SIZE], data2[SIZE], data3[SIZE], data4[SIZE];

    generate_random_unique(data1, SIZE);
    generate_sorted(data2);
    generate_reverse_sorted(data3);
    generate_formula(data4);

    run_experiment(data1, "무작위 정수");
    run_experiment(data2, "정렬된 정수");
    run_experiment(data3, "역정렬된 정수");
    run_experiment(data4, "공식 기반 정수");

    return 0;
}
