import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Scanner;

public class Trabalhador {
    private String nome;
    private String cpf;
    private double salarioBruto;
    private String cep;

    public Trabalhador(String nome, String cpf, double salarioBruto, String cep) {
        this.nome = nome;
        this.cpf = cpf;
        this.salarioBruto = salarioBruto;
        this.cep = cep;
    }

    public double calcularImpostoRenda() {
        return salarioBruto * 0.075;
    }

    public double calcularSalarioLiquido() {
        double impostoRenda = calcularImpostoRenda();
        return salarioBruto - impostoRenda;
    }

    public String consultarEndereco() {
        try {
            URL url = new URL("https://viacep.com.br/ws/" + cep + "/json/");
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            connection.connect();

            int responseCode = connection.getResponseCode();
            if (responseCode == 200) {
                Scanner scanner = new Scanner(url.openStream());
                StringBuilder endereco = new StringBuilder();
                while (scanner.hasNext()) {
                    endereco.append(scanner.nextLine());
                }
                scanner.close();
                return endereco.toString();
            } else {
                return "Erro ao consultar o endereço. Código de resposta: " + responseCode;
            }
        } catch (IOException e) {
            return "Erro ao consultar o endereço: " + e.getMessage();
        }
    }

    public boolean validarCPF() {
        String cpfNumerico = cpf.replaceAll("[^0-9]", "");

        if (cpfNumerico.length() != 11) {
            return false;
        }

        if (cpfNumerico.matches("(\\d)\\1{10}")) {
            return false;
        }

        int soma = 0;
        for (int i = 0; i < 9; i++) {
            soma += Character.getNumericValue(cpfNumerico.charAt(i)) * (10 - i);
        }
        int resto = soma % 11;
        int digito1 = (resto < 2) ? 0 : (11 - resto);

        soma = 0;
        for (int i = 0; i < 10; i++) {
            soma += Character.getNumericValue(cpfNumerico.charAt(i)) * (11 - i);
        }
        resto = soma % 11;
        int digito2 = (resto < 2) ? 0 : (11 - resto);

        return Character.getNumericValue(cpfNumerico.charAt(9)) == digito1 &&
               Character.getNumericValue(cpfNumerico.charAt(10)) == digito2;
    }

    public void salvarDadosEmArquivo(String nomeArquivo) {
        try {
            FileWriter arquivo = new FileWriter(nomeArquivo);
            PrintWriter gravarArquivo = new PrintWriter(arquivo);
            gravarArquivo.println("Nome: " + nome);
            gravarArquivo.println("CPF: " + cpf);
            gravarArquivo.println("Salário Bruto: " + salarioBruto);
            gravarArquivo.println("CEP: " + cep);
            arquivo.close();
            System.out.println("Dados salvos em " + nomeArquivo);
        } catch (IOException e) {
            System.out.println("Erro ao salvar arquivo: " + e.getMessage());
        }
    }
}
