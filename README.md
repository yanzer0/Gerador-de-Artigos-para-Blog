# Gerador-de-Artigos-para-Blog

# Importando Cohere
    import cohere

# Inicializar o cliente Cohere com sua chave de API
    co = cohere.Client('api_key')

    def gerar_artigo_cohere(topico, num_artigos=1):
        artigos = []
        for _ in range(num_artigos):
# Prompt mais detalhado para gerar um artigo completo com múltiplos parágrafos
            prompt = (f"Escreva um artigo detalhado e com vários parágrafos sobre o tema '{topico}'. "
                      "O artigo deve incluir uma introdução, vários pontos de discussão, e uma conclusão. "
                      "Cada ponto deve ser desenvolvido em parágrafos distintos. "
                      "Escreva ao menos 500 palavras no total.")
        
        response = co.generate(
            model='command-xlarge-nightly',  # Modelo grande para melhor performance
            prompt=prompt,
            max_tokens=1500,  # Aumentar o número de tokens para textos mais longos
            temperature=0.7,  # Controlar a criatividade
            num_generations=1  # Gerar um artigo por vez
        )
        artigo = response.generations[0].text.strip()
        artigos.append(artigo)
    return artigos

# Exemplo de uso
    if __name__ == "__main__":
        topico = "inteligência artificial"
        num_artigos = 3
        artigos = gerar_artigo_cohere(topico, num_artigos)

 # Salvar os artigos gerados em arquivos de texto
        for i, artigo in enumerate(artigos, 1):
            with open(f'artigo_{i}.txt', 'w', encoding='utf-8') as f:
                f.write(artigo)
            print(f"Artigo {i} salvo como artigo_{i}.txt")

