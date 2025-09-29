#include <bits/stdc++.h>
#include <functional>
using namespace std;

// cek vokal (lower & upper)
bool isVowel(char c) {
    c = tolower(c);
    return c=='a' || c=='e' || c=='i' || c=='o' || c=='u';
}

// reverse string tanpa menggunakan std::reverse
string myReverse(const string &s) {
    int n = (int)s.size();
    string t = s;
    for (int i = 0; i < n/2; ++i) {
        char tmp = t[i];
        t[i] = t[n-1-i];
        t[n-1-i] = tmp;
    }
    return t;
}

// encode sesuai mesin Timothy
string encode(const string &word) {
    if (word.empty()) return "";

    // 1) simpan ascii dari huruf pertama (sebagai int)
    int asciiFirst = (int)word[0];

    // 2) hapus semua vokal -> hasil hanya konsonan (preserve case)
    string cons = "";
    for (char c : word) {
        if (!isVowel(c)) cons.push_back(c);
    }

    // 3) balik konsonan (manual)
    string revCons = myReverse(cons);

    // 4) insert ascii string di tengah: left size = ceil(len/2)
    int L = (int)revCons.size();
    int leftSize = (L + 1) / 2; // ceil
    string left = revCons.substr(0, leftSize);
    string right = revCons.substr(leftSize);

    string asciiStr = to_string(asciiFirst); 
    return left + asciiStr + right;
}

// mencoba mendekode (mencari kandidat kata asli)
vector<string> decodeCandidates(const string &password, int maxVowelInsertions = 4) {
    vector<string> results;

    // 1) temukan block digit (asumsi ada tepat satu block berisi digit)
    int n = (int)password.size();
    int start = -1, end = -1;
    for (int i = 0; i < n; ++i) {
        if (isdigit(password[i])) {
            if (start == -1) start = i;
            end = i;
        }
    }
    if (start == -1) return results;

    string digitBlock = password.substr(start, end - start + 1);
    int asciiCode = stoi(digitBlock);
    char firstChar = (char)asciiCode;

    // 2) bentuk revCons (hapus digitBlock dari password)
    string revCons = password.substr(0, start) + password.substr(end+1);

    // 3) dapatkan skeleton konsonan (membalik kembali)
    string consSkeleton = myReverse(revCons);

    // 4) generate kandidat dengan sisipan vokal
    int m = (int)consSkeleton.size();
    vector<char> vowels = {'a','e','i','o','u','A','E','I','O','U'};

    function<void(int,string,int)> backtrack = [&](int pos, string built, int usedVowels) {
        if (pos == m) {
            if (!built.empty() && built[0] == firstChar) {
                string enc = encode(built);
                if (enc == password && 
                    find(results.begin(), results.end(), built) == results.end()) {
                    results.push_back(built);
                }
            }
            return;
        }

        // tanpa vokal
        backtrack(pos+1, built + consSkeleton[pos], usedVowels);

        // dengan vokal sebelum huruf
        if (usedVowels < maxVowelInsertions) {
            for (char v : vowels) {
                backtrack(pos+1, built + v + consSkeleton[pos], usedVowels + 1);
            }
        }

        // vokal di akhir kata
        if (pos == m-1 && usedVowels < maxVowelInsertions) {
            for (char v : vowels) {
                string cand = built + consSkeleton[pos] + v;
                if (!cand.empty() && cand[0] == firstChar) {
                    string enc = encode(cand);
                    if (enc == password && 
                        find(results.begin(), results.end(), cand) == results.end()) {
                        results.push_back(cand);
                    }
                }
            }
        }
    };

    backtrack(0, "", 0);
    return results;
}

int main() {
    cout << "Pilih mode: 1=Encode, 2=Decode\n";
    int mode;
    cin >> mode;

    if (mode == 1) {
        string w;
        cout << "Masukkan kata: ";
        cin >> w;
        cout << "Encoded: " << encode(w) << endl;
    } else if (mode == 2) {
        string pwd;
        cout << "Masukkan sandi: ";
        cin >> pwd;
        vector<string> candidates = decodeCandidates(pwd, 4);
        if (candidates.empty()) {
            cout << "Tidak ada kandidat ditemukan.\n";
        } else {
            cout << "Kemungkinan kata asli:\n";
            for (auto &s : candidates) cout << "- " << s << endl;
        }
    } else {
        cout << "Mode tidak dikenal.\n";
    }

    return 0;
}

# Program-CPP-LaurenciaZaharani_Deimos_TLS25
