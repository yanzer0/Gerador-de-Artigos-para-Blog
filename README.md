# Importando bibliotecas necessárias
    import cohere
    import tkinter as tk
    from tkinter import messagebox, filedialog
    from tkinter import ttk  # Para a barra de progresso e widgets estilizados
    import threading

# Inicializar o cliente Cohere com sua chave de API
    co = cohere.Client('api-key')

# Limites de tokens conhecidos (ajuste esses valores com base na sua conta)
    LIMITE_DIARIO = 20000  # Exemplo: 20.000 tokens diários

# Função para gerar artigos
    def gerar_artigo_cohere(topico, num_artigos=1, num_palavras=500, progress_callback=None):
        artigos = []
        for i in range(num_artigos):
            prompt = (f"Escreva um artigo detalhado e com vários parágrafos sobre o tema '{topico}'. "
                      "O artigo deve incluir uma introdução, vários pontos de discussão, e uma conclusão. "
                      "Cada ponto deve ser desenvolvido em parágrafos distintos. "
                      f"Escreva ao menos {num_palavras} palavras no total.")
        
        response = co.generate(
            model='command-xlarge-nightly',
            prompt=prompt,
            max_tokens=1500,
            temperature=0.7,
            num_generations=1
        )
        artigo = response.generations[0].text.strip()
        artigos.append(artigo)

        if progress_callback:
            progress_callback(i + 1, num_artigos)
    
    return artigos

# Função para salvar artigos
    def salvar_artigos(artigos, pasta_saida):
        for i, artigo in enumerate(artigos, 1):
            file_path = f"{pasta_saida}/artigo_{i}.txt"
            with open(file_path, 'w', encoding='utf-8') as f:
                f.write(artigo)
            messagebox.showinfo("Sucesso", f"Artigo {i} salvo como {file_path}")

# Função acionada ao clicar no botão (executada em uma thread)
    def gerar_e_salvar():
        topico = entry_topico.get()
        try:
            num_artigos = int(entry_num_artigos.get())
            num_palavras = int(entry_num_palavras.get())
        except ValueError:
            messagebox.showerror("Erro", "Insira valores válidos para os artigos e palavras.")
            return

        if not topico or num_artigos <= 0 or num_palavras <= 0:
            messagebox.showerror("Erro", "Preencha todos os campos corretamente.")
            return
    
        pasta_saida = filedialog.askdirectory(title="Selecione a pasta de saída")
        if not pasta_saida:
            messagebox.showerror("Erro", "Selecione uma pasta de saída válida.")
            return

        label_loading.config(text="Gerando artigos, por favor aguarde...")
        progress_bar['value'] = 0  # Reseta a barra de progresso
        janela.update()

        def atualizar_progresso(atual, total):
            progress_bar['value'] = (atual / total) * 100
            janela.update()

        artigos = gerar_artigo_cohere(topico, num_artigos, num_palavras, progress_callback=atualizar_progresso)
        salvar_artigos(artigos, pasta_saida)

        label_loading.config(text="")
        progress_bar['value'] = 0  # Limpa a barra de progresso
        messagebox.showinfo("Concluído", "Geração de artigos concluída!")

# Função para exibir o limite diário de tokens
    def exibir_limite_tokens():
        label_limite_tokens.config(text=f"Limite diário de palavras: {LIMITE_DIARIO}")

    def iniciar_geracao():
        thread = threading.Thread(target=gerar_e_salvar)
        thread.start()

# Criar a janela principal
    janela = tk.Tk()
    janela.title("Gerador de Artigos")
    janela.geometry("400x400")
    janela.configure(bg="#f0f0f0")  # Cor de fundo

# Centralizar janela
    janela.eval('tk::PlaceWindow . center')

# Definir o estilo dos widgets
    style = ttk.Style()
    style.theme_use("clam")  # Tema mais moderno
    style.configure("TLabel", background="#f0f0f0", font=("Helvetica", 10))
    style.configure("TButton", font=("Helvetica", 10), padding=5)
    style.configure("TEntry", padding=5)

# Estilizando a barra de progresso
    style.configure("TProgressbar", thickness=20, troughcolor="#e0e0e0", background="#4CAF50")

# Criar os elementos da interface
    label_topico = ttk.Label(janela, text="Tópico:")
    label_topico.pack(pady=5)

    entry_topico = ttk.Entry(janela, width=50)
    entry_topico.pack(pady=5)

    label_num_artigos = ttk.Label(janela, text="Número de Artigos:")
    label_num_artigos.pack(pady=5)

    entry_num_artigos = ttk.Entry(janela, width=10)
    entry_num_artigos.pack(pady=5)

    label_num_palavras = ttk.Label(janela, text="Número médio de palavras por artigo:")
    label_num_palavras.pack(pady=5)

    entry_num_palavras = ttk.Entry(janela, width=10)
    entry_num_palavras.pack(pady=5)

    botao_gerar = ttk.Button(janela, text="Gerar Artigos", command=iniciar_geracao)
    botao_gerar.pack(pady=10)

# Limite diário de tokens
    label_limite_tokens = ttk.Label(janela, text="")
    label_limite_tokens.pack(pady=5)
    exibir_limite_tokens()

# Label para mostrar o status de carregamento
    label_loading = ttk.Label(janela, text="", foreground="blue")
    label_loading.pack(pady=5)

# Barra de progresso
    progress_bar = ttk.Progressbar(janela, orient="horizontal", length=300, mode="determinate")
    progress_bar.pack(pady=10)

# Iniciar a aplicação
    janela.mainloop()
