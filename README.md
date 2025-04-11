# BigVigenere
Codigo BigVigenere, Laboratorio 1 EDA



    import java.io.*;
    import java.util.Scanner;

    public class BigVigenere {
    private int[] key;
    private final char[][] alphabet;

    public BigVigenere() {
        this.key = new int[]{0};
        this.alphabet = generateAlphabet();
    }

    public BigVigenere(String numericKey) {
        if (numericKey == null || numericKey.isEmpty()) {
            throw new IllegalArgumentException("La clave es invalida");
        }
        this.key = new int[numericKey.length()];
        for (int i = 0; i < numericKey.length(); i++) {
            this.key[i] = Character.getNumericValue(numericKey.charAt(i));
        }
        this.alphabet = generateAlphabet();
    }

    private char[][] generateAlphabet() {
        String caracteres = "abcdefghijklmnñopqrstuvwxyzABCDEFGHIJKLMNÑOPQRSTUVWXYZ0123456789";
        int largo = caracteres.length();
        char[][] alphabet = new char[largo][largo];

        for (int i = 0; i < largo; ++i) {
            for (int j = 0; j < largo; ++j) {
                alphabet[i][j] = caracteres.charAt((i + j) % largo);
            }
        }
        return alphabet;
    }

    public String encrypt(String message) {
        if (message == null) return "";

        StringBuilder encryptedMessage = new StringBuilder();
        int xd = 0;
        for (int i = 0; i < message.length(); i++) {
            char cActual = message.charAt(i);

            if (cActual == ' ' || cActual == ',' || cActual == '*' || cActual == '.' || cActual == ';' || cActual == ':' || cActual == '!' || cActual == '?') {
                encryptedMessage.append(cActual);
                continue;
            }

            int pos = valorNumerico(cActual);
            if (pos == -1) {
                encryptedMessage.append(cActual);
                continue;
            }

            int k = (key[xd] + 54) % 64;
            encryptedMessage.append(alphabet[k][pos]);

            xd = (xd + 1) % key.length;
        }
        return encryptedMessage.toString();
    }

    public String decrypt(String encryptedMessage) {
        if (encryptedMessage == null) {
            return "";
        }

        StringBuilder messageDesencriptado = new StringBuilder();
        int xd2 = 0;
        for (int i = 0; i < encryptedMessage.length(); i++) {
            char cActual = encryptedMessage.charAt(i);

            if (cActual == ' ' || cActual == '*' || cActual == '.' || cActual == ',' || cActual == ';' || cActual == ':' || cActual == '!' || cActual == '?') {
                messageDesencriptado.append(cActual);
                continue;
            }

            int k = (key[xd2] + 54) % 64;
            boolean encontro = false;
            for (int j = 0; j < 64; j++) {
                if (cActual == alphabet[k][j]) {
                    messageDesencriptado.append(alphabet[0][j]);
                    encontro = true;
                    break;
                }
            }
            if (!encontro) {
                messageDesencriptado.append(cActual);
            }
            xd2 = (xd2 + 1) % key.length;
        }
        return messageDesencriptado.toString();
    }

    public void reEncrypt() {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Mensaje encriptado: ");
        String encryptedMessage = scanner.nextLine();
        String messageDesencriptado = this.decrypt(encryptedMessage);
        System.out.println("Mensaje descifrado: " + messageDesencriptado);
        System.out.print("Nueva clave: ");
        String nKey = scanner.nextLine();

        if (nKey == null || nKey.isEmpty()) {
            System.out.println("La nueva clave es invalida. No se realizaron cambios.");
            return;
        }

        BigVigenere newVigenere = new BigVigenere(nKey);
        String nEncryptedMessage = newVigenere.encrypt(messageDesencriptado);

        this.key = newVigenere.key;

        System.out.println("Nuevo mensaje cifrado: " + nEncryptedMessage);
    }

    private int valorNumerico(char c) {
        String chars = "ABCDEFGHIJKLMNÑOPQRSTUVWXYZabcdefghijklmnñopqrstuvwxyz0123456789";
        return chars.indexOf(c);
    }

    public char search(int position) {
        for (int i = 0; i < alphabet.length; ++i) {
            for (int j = 0; j < alphabet.length; ++j) {
                if ((i * alphabet.length) + j == position) {
                    return alphabet[i][j];
                }
            }
        }
        return '\0';
    }

    public char optimalSearch(int position) {
        return alphabet[position / alphabet.length][position % alphabet.length];
    }

    public void print() { //esto nomas es para el reEncrypt
        for (int i = 0; i < alphabet.length; ++i) {
            for (int j = 0; j < alphabet.length; ++j) {
                System.out.print(alphabet[i][j] + " ");
            }
            System.out.println();
        }
    }

    public void MedirTiempo(String message) {
        long startEncrypt = System.nanoTime();
        String encrypted = encrypt(message);
        long endEncrypt = System.nanoTime();
        long startDecrypt = System.nanoTime();
        String decrypted = decrypt(encrypted);
        long endDecrypt = System.nanoTime();
        System.out.println("Tiempo de cifrado: " + (endEncrypt - startEncrypt) / 1e6 + " ms");
        System.out.println("Tiempo de descifrado: " + (endDecrypt - startDecrypt) / 1e6 + " ms");
    }

    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
             BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out))) {

            bw.write("Clave: ");
            bw.flush();
            String numericKey = br.readLine();

            BigVigenere vigenere = new BigVigenere(numericKey);

            bw.write("Mensaje a cifrar: ");
            bw.flush();
            String message = br.readLine();

            vigenere.MedirTiempo(message);

            String encryptedMessage = vigenere.encrypt(message);
            bw.write("Mensaje cifrado: " + encryptedMessage + "\n");

            String messageDesencriptado = vigenere.decrypt(encryptedMessage);
            bw.write("Mensaje descifrado: " + messageDesencriptado + "\n");

            bw.write("re-encriptar con una nueva clave? (s para si, n para no): ");
            bw.flush();
            String opcion = br.readLine();

            if (opcion.equalsIgnoreCase("s")) {
                vigenere.reEncrypt();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
            }
