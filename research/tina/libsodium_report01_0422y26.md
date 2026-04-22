# libsodium 使用報告

**日期：** 2026-04-22  
**作者：** Tina ✨

---

## 目錄

1. [簡介](#簡介)
2. [對稱加密使用方式](#1-對稱加密使用方式)
3. [非對稱加密使用方式](#2-非對稱加密使用方式)
4. [C++ Wrapper Class 製作](#3-c-wrapper-class-製作)
5. [參考資料](#參考資料)

---

## 簡介

**libsodium** 是一個現代化、易於使用的密碼學函式庫，基於 NaCl (Networking and Cryptography library) 開發。它提供：

- 高品質的加密原语
- 防止常見的安全漏洞（timing attacks、side-channel attacks）
- 簡潔一致的 API
- 跨平台支援

**安裝方式：**

```bash
# Ubuntu/Debian
sudo apt-get install libsodium-dev

# macOS
brew install libsodium

# 從原始碼編譯
git clone https://github.com/jedisct1/libsodium.git
cd libsodium
./configure && make && sudo make install
```

---

## 1. 對稱加密使用方式

對稱加密使用相同的金鑰進行加密與解密。libsodium 提供 `crypto_secretbox` 系列函式，使用 **XSalsa20-Poly1305** 演算法。

### 1.1 基本概念

- **金鑰長度：** `crypto_secretbox_KEYBYTES` (32 bytes)
- **Nonce 長度：** `crypto_secretbox_NONCEBYTES` (24 bytes)
- **MAC 長度：** `crypto_secretbox_MACBYTES` (16 bytes)

### 1.2 完整範例

```c
#include <stdio.h>
#include <string.h>
#include <sodium.h>

int main(void) {
    // 初始化 libsodium
    if (sodium_init() < 0) {
        fprintf(stderr, "Failed to initialize libsodium\n");
        return 1;
    }

    // ========== 金鑰生成 ==========
    unsigned char key[crypto_secretbox_KEYBYTES];
    crypto_secretbox_keygen(key);

    // ========== Nonce 生成 ==========
    unsigned char nonce[crypto_secretbox_NONCEBYTES];
    randombytes_buf(nonce, sizeof(nonce));

    // ========== 待加密訊息 ==========
    const char *message = "Hello, libsodium! This is a secret message.";
    unsigned long long message_len = strlen(message);

    // 密文緩衝區（需要額外空間存放 MAC）
    unsigned char ciphertext[crypto_secretbox_MACBYTES + message_len];

    // ========== 加密 ==========
    crypto_secretbox_easy(ciphertext, 
                          (const unsigned char *)message, 
                          message_len,
                          nonce, 
                          key);

    printf("加密成功！密文長度: %zu bytes\n", sizeof(ciphertext));

    // ========== 解密 ==========
    unsigned char decrypted[message_len + 1]; // +1 for null terminator

    if (crypto_secretbox_open_easy(decrypted, 
                                   ciphertext, 
                                   sizeof(ciphertext),
                                   nonce, 
                                   key) != 0) {
        fprintf(stderr, "解密失敗：訊息可能被竄改\n");
        return 1;
    }

    decrypted[message_len] = '\0';
    printf("解密成功：%s\n", decrypted);

    return 0;
}
```

### 1.3 編譯指令

```bash
gcc -o secretbox_example secretbox_example.c -lsodium
```

### 1.4 分離式加密（Detached Mode）

如果需要將 MAC 與密文分開存放：

```c
// 加密（MAC 分離）
unsigned char ciphertext[message_len];
unsigned char mac[crypto_secretbox_MACBYTES];

crypto_secretbox_detached(ciphertext, mac,
                          (const unsigned char *)message, message_len,
                          nonce, key);

// 解密（MAC 分離）
unsigned char decrypted[message_len + 1];

if (crypto_secretbox_open_detached(decrypted, ciphertext, mac,
                                   message_len, nonce, key) != 0) {
    // 解密失敗
}
```

---

## 2. 非對稱加密使用方式

libsodium 提供多種非對稱加密方案，最著名的是 **X25519**（金鑰交換）和 **Ed25519**（數位簽章）。

### 2.1 金鑰交換（X25519）

使用 Curve25519 橢圓曲線進行 Diffie-Hellman 金鑰交換。

```c
#include <stdio.h>
#include <string.h>
#include <sodium.h>

int main(void) {
    if (sodium_init() < 0) {
        return 1;
    }

    // ========== Alice 金鑰對 ==========
    unsigned char alice_sk[crypto_scalarmult_curve25519_SCALARBYTES];
    unsigned char alice_pk[crypto_scalarmult_curve25519_BYTES];
    crypto_scalarmult_curve25519_base(alice_pk, alice_sk);

    // ========== Bob 金鑰對 ==========
    unsigned char bob_sk[crypto_scalarmult_curve25519_SCALARBYTES];
    unsigned char bob_pk[crypto_scalarmult_curve25519_BYTES];
    crypto_scalarmult_curve25519_base(bob_pk, bob_sk);

    // ========== 金鑰交換 ==========
    unsigned char alice_shared[crypto_scalarmult_curve25519_BYTES];
    unsigned char bob_shared[crypto_scalarmult_curve25519_BYTES];

    // Alice 計算共享金鑰
    if (crypto_scalarmult_curve25519(alice_shared, alice_sk, bob_pk) != 0) {
        fprintf(stderr, "金鑰交換失敗\n");
        return 1;
    }

    // Bob 計算共享金鑰
    if (crypto_scalarmult_curve25519(bob_shared, bob_sk, alice_pk) != 0) {
        fprintf(stderr, "金鑰交換失敗\n");
        return 1;
    }

    // 驗證雙方得到相同的共享金鑰
    if (memcmp(alice_shared, bob_shared, sizeof(alice_shared)) == 0) {
        printf("✓ 金鑰交換成功！雙方得到相同的共享金鑰\n");
    }

    // 可以使用共享金鑰進行對稱加密
    // 注意：建議使用 KDF（如 crypto_kdf_derive_from_key）衍生實際使用的金鑰

    return 0;
}
```

### 2.2 公鑰加密（crypto_box）

使用 X25519 + XSalsa20-Poly1305 進行公鑰加密。

```c
#include <stdio.h>
#include <string.h>
#include <sodium.h>

int main(void) {
    if (sodium_init() < 0) {
        return 1;
    }

    // ========== 接收方金鑰對 ==========
    unsigned char receiver_pk[crypto_box_PUBLICKEYBYTES];
    unsigned char receiver_sk[crypto_box_SECRETKEYBYTES];
    crypto_box_keypair(receiver_pk, receiver_sk);

    // ========== 發送方加密 ==========
    const char *message = "這是機密訊息，只有接收方能解密";
    unsigned long long message_len = strlen(message);

    unsigned char nonce[crypto_box_NONCEBYTES];
    randombytes_buf(nonce, sizeof(nonce));

    // 密文緩衝區
    unsigned char ciphertext[crypto_box_MACBYTES + message_len];

    // 加密（發送方需知道接收方的公鑰）
    if (crypto_box_easy(ciphertext, 
                       (const unsigned char *)message, 
                       message_len,
                       nonce, 
                       receiver_pk, 
                       receiver_sk) != 0) {
        fprintf(stderr, "加密失敗\n");
        return 1;
    }

    printf("✓ 公鑰加密成功\n");

    // ========== 接收方解密 ==========
    unsigned char decrypted[message_len + 1];

    if (crypto_box_open_easy(decrypted, 
                            ciphertext, 
                            sizeof(ciphertext),
                            nonce, 
                            receiver_pk, 
                            receiver_sk) != 0) {
        fprintf(stderr, "解密失敗\n");
        return 1;
    }

    decrypted[message_len] = '\0';
    printf("✓ 解密成功：%s\n", decrypted);

    return 0;
}
```

### 2.3 數位簽章（Ed25519）

使用 Ed25519 進行訊息簽章與驗證。

```c
#include <stdio.h>
#include <string.h>
#include <sodium.h>

int main(void) {
    if (sodium_init() < 0) {
        return 1;
    }

    // ========== 簽署者金鑰對 ==========
    unsigned char pk[crypto_sign_PUBLICKEYBYTES];
    unsigned char sk[crypto_sign_SECRETKEYBYTES];
    crypto_sign_keypair(pk, sk);

    // ========== 待簽署訊息 ==========
    const char *message = "這份文件由我簽署，內容未被竄改";
    unsigned long long message_len = strlen(message);

    // ========== 簽署 ==========
    unsigned char sig[crypto_sign_BYTES];

    crypto_sign_detached(sig, NULL, 
                        (const unsigned char *)message, 
                        message_len, 
                        sk);

    printf("✓ 簽章完成，簽章長度: %zu bytes\n", sizeof(sig));

    // ========== 驗證 ==========
    if (crypto_sign_verify_detached(sig, 
                                   (const unsigned char *)message, 
                                   message_len, 
                                   pk) == 0) {
        printf("✓ 簽章驗證成功\n");
    } else {
        printf("✗ 簽章驗證失敗\n");
    }

    return 0;
}
```

---

## 3. C++ Wrapper Class 製作

以下是一個 RAII 風格的 C++ wrapper class，封裝對稱加密功能。

### 3.1 完整範例

```cpp
// sodium_wrapper.hpp
#ifndef SODIUM_WRAPPER_HPP
#define SODIUM_WRAPPER_HPP

#include <string>
#include <vector>
#include <stdexcept>
#include <sodium.h>

namespace crypto {

/**
 * @brief libsodium 初始化管理（Singleton）
 */
class SodiumInit {
public:
    static SodiumInit& instance() {
        static SodiumInit inst;
        return inst;
    }

    bool isInitialized() const { return initialized_; }

private:
    SodiumInit() {
        if (sodium_init() < 0) {
            throw std::runtime_error("Failed to initialize libsodium");
        }
        initialized_ = true;
    }

    SodiumInit(const SodiumInit&) = delete;
    SodiumInit& operator=(const SodiumInit&) = delete;

    bool initialized_ = false;
};

/**
 * @brief 對稱加密包裝類別
 * 
 * 使用 XSalsa20-Poly1305 演算法
 */
class SecretBox {
public:
    static constexpr size_t KEY_SIZE = crypto_secretbox_KEYBYTES;
    static constexpr size_t NONCE_SIZE = crypto_secretbox_NONCEBYTES;
    static constexpr size_t MAC_SIZE = crypto_secretbox_MACBYTES;

    using Key = std::vector<unsigned char>;
    using Nonce = std::vector<unsigned char>;

    /**
     * @brief 建構函式
     */
    SecretBox() {
        SodiumInit::instance(); // 確保 libsodium 已初始化
    }

    /**
     * @brief 生成隨機金鑰
     */
    Key generateKey() const {
        Key key(KEY_SIZE);
        crypto_secretbox_keygen(key.data());
        return key;
    }

    /**
     * @brief 生成隨機 Nonce
     */
    Nonce generateNonce() const {
        Nonce nonce(NONCE_SIZE);
        randombytes_buf(nonce.data(), NONCE_SIZE);
        return nonce;
    }

    /**
     * @brief 加密字串
     * 
     * @param plaintext 明文
     * @param nonce Nonce
     * @param key 金鑰
     * @return 密文（包含 MAC）
     */
    std::vector<unsigned char> encrypt(
        const std::string& plaintext,
        const Nonce& nonce,
        const Key& key
    ) const {
        return encrypt(
            reinterpret_cast<const unsigned char*>(plaintext.data()),
            plaintext.size(),
            nonce,
            key
        );
    }

    /**
     * @brief 加密二進位資料
     */
    std::vector<unsigned char> encrypt(
        const unsigned char* plaintext,
        size_t plaintext_len,
        const Nonce& nonce,
        const Key& key
    ) const {
        validateKey(key);
        validateNonce(nonce);

        std::vector<unsigned char> ciphertext(MAC_SIZE + plaintext_len);

        crypto_secretbox_easy(
            ciphertext.data(),
            plaintext,
            plaintext_len,
            nonce.data(),
            key.data()
        );

        return ciphertext;
    }

    /**
     * @brief 解密
     * 
     * @param ciphertext 密文
     * @param nonce Nonce
     * @param key 金鑰
     * @return 明文
     * @throws std::runtime_error 解密失敗
     */
    std::string decryptToString(
        const std::vector<unsigned char>& ciphertext,
        const Nonce& nonce,
        const Key& key
    ) const {
        auto plaintext = decrypt(ciphertext, nonce, key);
        return std::string(plaintext.begin(), plaintext.end());
    }

    /**
     * @brief 解密為二進位資料
     */
    std::vector<unsigned char> decrypt(
        const std::vector<unsigned char>& ciphertext,
        const Nonce& nonce,
        const Key& key
    ) const {
        validateKey(key);
        validateNonce(nonce);

        if (ciphertext.size() < MAC_SIZE) {
            throw std::runtime_error("Ciphertext too short");
        }

        size_t plaintext_len = ciphertext.size() - MAC_SIZE;
        std::vector<unsigned char> plaintext(plaintext_len);

        if (crypto_secretbox_open_easy(
            plaintext.data(),
            ciphertext.data(),
            ciphertext.size(),
            nonce.data(),
            key.data()
        ) != 0) {
            throw std::runtime_error("Decryption failed: invalid ciphertext or key");
        }

        return plaintext;
    }

    /**
     * @brief 將二進位資料轉換為十六進位字串
     */
    static std::string toHex(const std::vector<unsigned char>& data) {
        std::string hex(data.size() * 2, '\0');
        sodium_bin2hex(hex.data(), hex.size() + 1, data.data(), data.size());
        return hex;
    }

    /**
     * @brief 從十六進位字串解析二進位資料
     */
    static std::vector<unsigned char> fromHex(const std::string& hex) {
        std::vector<unsigned char> data(hex.size() / 2);
        if (sodium_hex2bin(
            data.data(), data.size(),
            hex.data(), hex.size(),
            nullptr, nullptr, nullptr
        ) != 0) {
            throw std::runtime_error("Invalid hex string");
        }
        return data;
    }

private:
    void validateKey(const Key& key) const {
        if (key.size() != KEY_SIZE) {
            throw std::runtime_error("Invalid key size");
        }
    }

    void validateNonce(const Nonce& nonce) const {
        if (nonce.size() != NONCE_SIZE) {
            throw std::runtime_error("Invalid nonce size");
        }
    }
};

/**
 * @brief 公鑰加密包裝類別
 */
class Box {
public:
    static constexpr size_t PUBLICKEY_SIZE = crypto_box_PUBLICKEYBYTES;
    static constexpr size_t SECRETKEY_SIZE = crypto_box_SECRETKEYBYTES;
    static constexpr size_t NONCE_SIZE = crypto_box_NONCEBYTES;
    static constexpr size_t MAC_SIZE = crypto_box_MACBYTES;

    using PublicKey = std::vector<unsigned char>;
    using SecretKey = std::vector<unsigned char>;
    using Nonce = std::vector<unsigned char>;

    Box() {
        SodiumInit::instance();
    }

    /**
     * @brief 生成金鑰對
     */
    std::pair<PublicKey, SecretKey> generateKeyPair() const {
        PublicKey pk(PUBLICKEY_SIZE);
        SecretKey sk(SECRETKEY_SIZE);
        crypto_box_keypair(pk.data(), sk.data());
        return {pk, sk};
    }

    /**
     * @brief 生成 Nonce
     */
    Nonce generateNonce() const {
        Nonce nonce(NONCE_SIZE);
        randombytes_buf(nonce.data(), NONCE_SIZE);
        return nonce;
    }

    /**
     * @brief 公鑰加密
     */
    std::vector<unsigned char> encrypt(
        const std::string& plaintext,
        const Nonce& nonce,
        const PublicKey& receiver_pk,
        const SecretKey& sender_sk
    ) const {
        validatePublicKey(receiver_pk);
        validateSecretKey(sender_sk);
        validateNonce(nonce);

        std::vector<unsigned char> ciphertext(MAC_SIZE + plaintext.size());

        if (crypto_box_easy(
            ciphertext.data(),
            reinterpret_cast<const unsigned char*>(plaintext.data()),
            plaintext.size(),
            nonce.data(),
            receiver_pk.data(),
            sender_sk.data()
        ) != 0) {
            throw std::runtime_error("Encryption failed");
        }

        return ciphertext;
    }

    /**
     * @brief 公鑰解密
     */
    std::string decrypt(
        const std::vector<unsigned char>& ciphertext,
        const Nonce& nonce,
        const PublicKey& sender_pk,
        const SecretKey& receiver_sk
    ) const {
        validatePublicKey(sender_pk);
        validateSecretKey(receiver_sk);
        validateNonce(nonce);

        if (ciphertext.size() < MAC_SIZE) {
            throw std::runtime_error("Ciphertext too short");
        }

        size_t plaintext_len = ciphertext.size() - MAC_SIZE;
        std::vector<unsigned char> plaintext(plaintext_len);

        if (crypto_box_open_easy(
            plaintext.data(),
            ciphertext.data(),
            ciphertext.size(),
            nonce.data(),
            sender_pk.data(),
            receiver_sk.data()
        ) != 0) {
            throw std::runtime_error("Decryption failed");
        }

        return std::string(plaintext.begin(), plaintext.end());
    }

private:
    void validatePublicKey(const PublicKey& pk) const {
        if (pk.size() != PUBLICKEY_SIZE) {
            throw std::runtime_error("Invalid public key size");
        }
    }

    void validateSecretKey(const SecretKey& sk) const {
        if (sk.size() != SECRETKEY_SIZE) {
            throw std::runtime_error("Invalid secret key size");
        }
    }

    void validateNonce(const Nonce& nonce) const {
        if (nonce.size() != NONCE_SIZE) {
            throw std::runtime_error("Invalid nonce size");
        }
    }
};

} // namespace crypto

#endif // SODIUM_WRAPPER_HPP
```

### 3.2 使用範例

```cpp
// main.cpp
#include <iostream>
#include "sodium_wrapper.hpp"

int main() {
    using namespace crypto;

    // ========== 對稱加密範例 ==========
    std::cout << "=== 對稱加密 (SecretBox) ===" << std::endl;

    SecretBox secretBox;

    auto key = secretBox.generateKey();
    auto nonce = secretBox.generateNonce();

    std::string message = "這是機密訊息！";
    std::cout << "原文: " << message << std::endl;

    auto ciphertext = secretBox.encrypt(message, nonce, key);
    std::cout << "密文 (hex): " << SecretBox::toHex(ciphertext).substr(0, 32) << "..." << std::endl;

    auto decrypted = secretBox.decryptToString(ciphertext, nonce, key);
    std::cout << "解密: " << decrypted << std::endl;

    std::cout << std::endl;

    // ========== 公鑰加密範例 ==========
    std::cout << "=== 公鑰加密 (Box) ===" << std::endl;

    Box box;

    // Alice 和 Bob 各自生成金鑰對
    auto [alice_pk, alice_sk] = box.generateKeyPair();
    auto [bob_pk, bob_sk] = box.generateKeyPair();

    auto box_nonce = box.generateNonce();

    std::string secret_msg = "Hello, Bob! 這是 Alice 的秘密訊息";
    std::cout << "Alice 發送: " << secret_msg << std::endl;

    // Alice 用 Bob 的公鑰加密
    auto encrypted = box.encrypt(secret_msg, box_nonce, bob_pk, alice_sk);

    // Bob 用自己的私鑰解密
    auto decrypted_msg = box.decrypt(encrypted, box_nonce, alice_pk, bob_sk);
    std::cout << "Bob 收到: " << decrypted_msg << std::endl;

    return 0;
}
```

### 3.3 編譯指令

```bash
g++ -std=c++17 -o sodium_wrapper_example main.cpp -lsodium
```

### 3.4 輸出結果

```
=== 對稱加密 (SecretBox) ===
原文: 這是機密訊息！
密文 (hex): a1b2c3d4e5f6...
解密: 這是機密訊息！

=== 公鑰加密 (Box) ===
Alice 發送: Hello, Bob! 這是 Alice 的秘密訊息
Bob 收到: Hello, Bob! 這是 Alice 的秘密訊息
```

---

## 參考資料

### 官方文件
- [libsodium 官方文件](https://doc.libsodium.org/)
- [libsodium GitHub Repository](https://github.com/jedisct1/libsodium)

### 參考專案

以下 GitHub 專案展示了 libsodium 的實際應用：

1. **libsodium 原始專案**
   - Repository: https://github.com/jedisct1/libsodium
   - 作者: Frank Denis
   - 說明: 官方維護，包含完整的 API 文件與範例

2. **NaCl 原始專案**
   - Repository: https://nacl.cr.yp.to/
   - 作者: Daniel J. Bernstein, Tanja Lange, Peter Schwabe
   - 說明: libsodium 的基礎，提供核心加密原語

3. **PyNaCl (Python 綁定)**
   - Repository: https://github.com/pyca/pynacl
   - 說明: Python 綁定的實作參考，展示了 RAII 封裝模式

4. ** libsodium-js (JavaScript 綁定)**
   - Repository: https://github.com/jedisct1/libsodium.js
   - 說明: WebAssembly 編譯範例

### 書籍與文章
- Bernstein, D. J., et al. "NaCl: Networking and Cryptography library"
- 《Serious Cryptography》 by Jean-Philippe Aumasson

---

## 附錄：常用 API 速查表

| 功能 | 函式 | 說明 |
|------|------|------|
| 初始化 | `sodium_init()` | 必須在其他函式之前呼叫 |
| 隨機位元組 | `randombytes_buf()` | 生成密碼學安全的隨機資料 |
| **對稱加密** | `crypto_secretbox_easy()` | XSalsa20-Poly1305 加密 |
| 對稱解密 | `crypto_secretbox_open_easy()` | 解密並驗證 |
| **公鑰加密** | `crypto_box_easy()` | X25519 + XSalsa20-Poly1305 |
| 公鑰解密 | `crypto_box_open_easy()` | |
| 金鑰對生成 | `crypto_box_keypair()` | |
| **簽章** | `crypto_sign_detached()` | Ed25519 簽章 |
| 簽章驗證 | `crypto_sign_verify_detached()` | |
| 金鑰交換 | `crypto_scalarmult_curve25519()` | X25519 DH |
| **Hash** | `crypto_generichash()` | BLAKE2b |
| 密碼 Hash | `crypto_pwhash()` | Argon2id |

---

*報告完成時間：2026-04-22 UTC*  
*作者：Tina ✨*
