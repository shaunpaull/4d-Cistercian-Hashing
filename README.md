# 4d-Cistercian-Hashing
4d cistercian hashing

#include <iostream>
#include <vector>
#include <random>
#include <bitset>
#include <sstream>
#include <iomanip>

// Structure to represent a lattice symbol with color and complexity
struct LatticeSymbol {
    unsigned int symbol;            // Unicode symbol
    std::vector<std::string> colors; // Colors for each dimension
    std::bitset<256> complexity;    // Complexity key
};

// Function to create a 4D lattice with colors and additional complexity
std::vector<std::vector<std::vector<std::vector<LatticeSymbol>>>> createLattice(int width, int height, int depth, int time) {
    // Create a random number generator
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<unsigned int> distribution(0, 1114111); // Maximum Unicode code point

    // Create the lattice structure with colors and complexity
    std::vector<std::vector<std::vector<std::vector<LatticeSymbol>>>> lattice(width, std::vector<std::vector<std::vector<LatticeSymbol>>>(height, std::vector<std::vector<LatticeSymbol>>(depth, std::vector<LatticeSymbol>(time))));

    // Fill the lattice with random Unicode symbols, colors, and complexity
    for (int i = 0; i < width; i++) {
        for (int j = 0; j < height; j++) {
            for (int k = 0; k < depth; k++) {
                for (int l = 0; l < time; l++) {
                    unsigned int symbol = distribution(gen);
                    int numColors = gen() % 10 + 1; // Random number of colors (1 to 10)
                    std::vector<std::string> colors(numColors);
                    for (int c = 0; c < numColors; c++) {
                        colors[c] = "Color" + std::to_string(c + 1);
                    }
                    std::bitset<256> complexity;
                    for (int b = 0; b < 256; b++) {
                        complexity[b] = gen() % 2; // Generate a random bit for each position in the 256-bit key
                    }

                    LatticeSymbol latticeSymbol;
                    latticeSymbol.symbol = symbol;
                    latticeSymbol.colors = colors;
                    latticeSymbol.complexity = complexity;

                    lattice[i][j][k][l] = latticeSymbol;
                }
            }
        }
    }

    return lattice;
}

// Function to encrypt a message using the 4D Cistercian lattice and custom encryption
std::string encryptMessage(const std::string& message, const std::vector<std::vector<std::vector<std::vector<LatticeSymbol>>>>& lattice, const std::string& encryptionKey, int numRounds) {
    std::vector<unsigned char> encryptedData;
    std::vector<unsigned char> key(encryptionKey.begin(), encryptionKey.end());

    for (int round = 0; round < numRounds; round++) {
        for (char c : message) {
            unsigned int index1 = c % lattice.size();
            unsigned int index2 = c / lattice.size() % lattice[0].size();
            unsigned int index3 = c / (lattice.size() * lattice[0].size()) % lattice[0][0].size();
            unsigned int index4 = c / (lattice.size() * lattice[0].size() * lattice[0][0].size()) % lattice[0][0][0].size();

            const LatticeSymbol& latticeSymbol = lattice[index1][index2][index3][index4];
            std::bitset<256> keyBits = latticeSymbol.complexity;

            // Find the most efficient Unicode symbol with the highest complexity
            unsigned int maxSymbol = latticeSymbol.symbol;
            unsigned int maxComplexity = keyBits.count();
            for (int i = 0; i < lattice.size(); i++) {
                for (int j = 0; j < lattice[0].size(); j++) {
                    for (int k = 0; k < lattice[0][0].size(); k++) {
                        for (int l = 0; l < lattice[0][0][0].size(); l++) {
                            const LatticeSymbol& symbol = lattice[i][j][k][l];
                            std::bitset<256> symbolComplexity = symbol.complexity;
                            unsigned int complexity = symbolComplexity.count();
                            if (complexity > maxComplexity) {
                                maxSymbol = symbol.symbol;
                                maxComplexity = complexity;
                            }
                        }
                    }
                }
            }

            std::vector<unsigned long long> keys(4);
            for (int j = 0; j < 4; j++) {
                std::bitset<64> subKey;
                for (int b = 0; b < 64; b++) {
                    subKey[b] = keyBits[b + (j * 64)];
                }
                keys[j] = subKey.to_ullong();
            }

            unsigned char encryptedChar = c ^ (key[0] & 0xFF) ^
                                          (keys[0] & 0xFFFFFFFF) & 0xFF ^
                                          (keys[1] & 0xFFFFFFFF) & 0xFF ^
                                          (keys[2] & 0xFFFFFFFF) & 0xFF ^
                                          (keys[3] & 0xFFFFFFFF) & 0xFF ^
                                          (maxSymbol & 0xFF);

            encryptedData.push_back(encryptedChar);
        }
    }

    // Convert the encrypted data to a hexadecimal string
    std::stringstream ss;
    ss << std::hex << std::setfill('0');
    for (unsigned char byte : encryptedData) {
        ss << std::setw(2) << static_cast<int>(byte);
    }

    return ss.str();
}

// Function to create a custom hash from a given input using the 4D Cistercian lattice and encryption
std::string createCustomHash(const std::string& input) {
    int width = 10;
    int height = 10;
    int depth = 10;
    int time = 10;

    // Create a 4D Cistercian lattice
    std::vector<std::vector<std::vector<std::vector<LatticeSymbol>>>> lattice = createLattice(width, height, depth, time);

    // Encrypt the input message using the lattice structure
    std::string encryptionKey = "SecretKey";
    int numRounds = 10;
    std::string encryptedMessage = encryptMessage(input, lattice, encryptionKey, numRounds);

    // Return the encrypted message as the custom hash
    return encryptedMessage;
}

int main() {
    // Obtain user input
    std::cout << "Enter a message to hash:" ;
    std::string input;
    std::getline(std::cin, input);

    // Create a custom hash from the input
    std::string customHash = createCustomHash(input);

    std::cout << "Custom hash: " << customHash << std::endl;

    return 0;
}
