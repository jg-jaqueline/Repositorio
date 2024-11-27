import ArgumentParser
import Foundation

// Função para ler o conteúdo de um arquivo
func lerArquivo(caminho: String) -> String? {
    do {
        let conteudo = try String(contentsOfFile: caminho, encoding: .utf8)
        return conteudo
    } catch {
        print("Erro ao ler o arquivo: \(error)")
        return nil
    }
}

// Função para contar o número de palavras no conteúdo do arquivo
func contarPalavras(conteudo: String) -> Int {
    let palavras = conteudo.split { !$0.isLetter }
    return palavras.count
}

// Função para contar as ocorrências de uma palavra no conteúdo do arquivo
func buscarPalavra(conteudo: String, palavra: String) -> Int {
    let palavras = conteudo.split { !$0.isLetter }
    return palavras.filter { $0.lowercased() == palavra.lowercased() }.count
}

// Função para substituir todas as ocorrências de uma palavra por outra
func substituirPalavra(conteudo: String, palavraAntiga: String, palavraNova: String) -> String {
    return conteudo.replacingOccurrences(of: palavraAntiga, with: palavraNova)
}

// Função para escrever o conteúdo alterado de volta no arquivo
func escreverNoArquivo(caminho: String, conteudo: String) {
    do {
        try conteudo.write(toFile: caminho, atomically: true, encoding: .utf8)
    } catch {
        print("Erro ao escrever no arquivo: \(error)")
    }
}

struct CLI: ParsableCommand {
    // Argumento para o caminho do arquivo
    @Argument(help: "Caminho do arquivo de texto")
    var caminho: String
    
    // Opção para buscar uma palavra específica
    @Option(name: .shortAndLong, help: "Palavra a ser buscada no arquivo")
    var buscar: String?
    
    // Opção para substituir uma palavra
    @Option(name: .shortAndLong, help: "Palavra para substituição")
    var substituir: String?
    
    func run() throws {
        // Tenta ler o conteúdo do arquivo
        guard let conteudo = lerArquivo(caminho: caminho) else {
            print("Erro ao ler o arquivo.")
            return
        }
        
        // Se houver uma palavra para buscar, conta suas ocorrências
        if let buscar = buscar {
            let ocorrencias = buscarPalavra(conteudo: conteudo, palavra: buscar)
            print("A palavra '\(buscar)' aparece \(ocorrencias) vezes.")
        }
        
        // Se houver uma palavra para substituir, realiza a substituição
        if let substituir = substituir {
            if let buscar = buscar {
                let conteudoAlterado = substituirPalavra(conteudo: conteudo, palavraAntiga: buscar, palavraNova: substituir)
                escreverNoArquivo(caminho: caminho, conteudo: conteudoAlterado)
                print("Conteúdo alterado. As palavras foram substituídas.")
            } else {
                print("É necessário especificar uma palavra para buscar antes de substituir.")
            }
        } else if buscar == nil {
            // Caso não haja opção de busca ou substituição, conta o número total de palavras
            let numeroDePalavras = contarPalavras(conteudo: conteudo)
            print("O arquivo contém \(numeroDePalavras) palavras.")
        }
    }
}

CLI.main()
