import tkinter as tk
from tkinter import ttk, messagebox, font, filedialog
import sqlite3
import os
import pandas as pd

# Configurando o caminho do banco de dados para a pasta raiz do Windows
db_path = os.path.join(os.path.expanduser("~"), "cadastro.db")
conn = sqlite3.connect(db_path)
cursor = conn.cursor()

# Criação da tabela no banco de dados
cursor.execute('''
    CREATE TABLE IF NOT EXISTS cadastro (
        nome TEXT,
        prontuario TEXT PRIMARY KEY,
        endereco TEXT
    )
''')

def add(event=None):
    nome = entrada_nome.get().upper()
    prontuario = entrada_prontuario.get()
    endereco = entrada_endereco.get().upper()

    if nome and prontuario and endereco:
        if prontuario.isdigit() and 1 <= int(prontuario) <= 20300:  # Altere a faixa de prontuário
            try:
                cursor.execute('INSERT INTO cadastro VALUES (?, ?, ?)', (nome, prontuario, endereco))
                conn.commit()
                messagebox.showinfo("Cadastro", "Cadastro realizado com sucesso!")
                entrada_nome.delete(0, tk.END)
                entrada_prontuario.delete(0, tk.END)
                entrada_endereco.delete(0, tk.END)
                update_table()
            except sqlite3.IntegrityError:
                messagebox.showerror("Erro", "Prontuário já existente!")
        else:
            messagebox.showerror("Erro", "Prontuário inválido! (1-20300)")  # Altere a faixa de prontuário
    else:
        messagebox.showerror("Erro", "Preencha todos os campos!")

def search():
    search = entrada_search.get().upper()
    tree.delete(*tree.get_children())
    if search:
        cursor.execute('SELECT * FROM cadastro WHERE nome LIKE ? OR endereco LIKE ? OR prontuario LIKE ?', 
                       ('%' + search + '%', '%' + search + '%', '%' + search + '%'))
        results = cursor.fetchall()
        if results:
            for result in results:
                tree.insert('', 'end', values=result)
        else:
            messagebox.showinfo("Resultado", "Nenhum registro encontrado!")
    else:
        update_table()

def update_table():
    tree.delete(*tree.get_children())
    cursor.execute('SELECT * FROM cadastro ORDER BY nome ASC')
    results = cursor.fetchall()
    for result in results:
        tree.insert('', 'end', values=(result[0].upper(), result[1], result[2].upper()))

def delete():
    selected_item = tree.selection()
    if selected_item:
        item = tree.item(selected_item)
        prontuario = item['values'][1]
        confirm = messagebox.askyesno("Confirmar", "Deseja deletar o cadastro selecionado?")
        if confirm:
            cursor.execute('DELETE FROM cadastro WHERE prontuario = ?', (prontuario,))
            conn.commit()
            tree.delete(selected_item)
            messagebox.showinfo("Deletado", "Cadastro deletado com sucesso!")
            update_table()
    else:
        messagebox.showerror("Erro", "Nenhum cadastro selecionado!")

def import_from_excel():
    file_path = filedialog.askopenfilename(
        title="Selecione o arquivo Excel",
        filetypes=(("Arquivos Excel", "*.xlsx *.xls"), ("Todos os arquivos", "*.*"))
    )
    if file_path:
        try:
            # Leitura do arquivo Excel
            df = pd.read_excel(file_path, engine='openpyxl')
            
            # Exibir os nomes das colunas para depuração
            print(f"Colunas encontradas no arquivo Excel: {df.columns.tolist()}")
            
            # Remover espaços extras dos nomes das colunas
            df.columns = df.columns.str.strip()
            
            # Exibir as colunas após remoção dos espaços extras para verificação
            print(f"Colunas após strip: {df.columns.tolist()}")

            # Verifica se as colunas necessárias existem no arquivo Excel
            required_columns = ['nome', 'prontuario', 'endereco']
            missing_columns = [col for col in required_columns if col not in df.columns]
            
            if missing_columns:
                messagebox.showerror("Erro", f"O arquivo Excel está faltando as colunas: {', '.join(missing_columns)}")
                return
            
            # Inserir dados no banco de dados
            for _, row in df.iterrows():
                nome = str(row['nome']).upper()
                prontuario = str(row['prontuario']).strip()
                endereco = str(row['endereco']).upper()
                
                if prontuario.isdigit() and 1 <= int(prontuario) <= 20300:  # Validação do prontuário
                    try:
                        cursor.execute('INSERT INTO cadastro VALUES (?, ?, ?)', (nome, prontuario, endereco))
                    except sqlite3.IntegrityError:
                        # Ignorar prontuários duplicados
                        continue
            
            conn.commit()
            messagebox.showinfo("Importação", "Dados importados com sucesso!")
            update_table()
        except Exception as e:
            messagebox.showerror("Erro", f"Erro ao importar dados do Excel: {e}")

# Criação da interface
janela = tk.Tk()
janela.title("Sistema de Cadastro")

# Configurando para que a janela inicial seja centralizada e ajustada ao tamanho da tela
screen_width = janela.winfo_screenwidth()
screen_height = janela.winfo_screenheight()
janela.geometry(f"{int(screen_width * 0.6)}x{int(screen_height * 0.6)}+{int(screen_width * 0.2)}+{int(screen_height * 0.2)}")
janela.minsize(800, 400)
janela.configure(bg="#ADD8E6")  # Azul bebê como cor de fundo

janela.grid_rowconfigure(4, weight=1)
janela.grid_columnconfigure(1, weight=1)

def ajustar_tamanho(event):
    largura = event.width
    altura = event.height
    tree.column('nome', width=int(largura * 0.4))
    tree.column('prontuario', width=int(largura * 0.2))
    tree.column('endereco', width=int(largura * 0.4))

janela.bind("<Configure>", ajustar_tamanho)

bold_font = font.Font(weight="bold", size=10)

# Labels e entradas
label_nome = tk.Label(janela, text="Nome:", font=bold_font, anchor="w", bg="#ADD8E6")
label_nome.grid(row=0, column=0, sticky="w", padx=10, pady=5)
entrada_nome = tk.Entry(janela, font=("Arial", 12), width=30)
entrada_nome.grid(row=0, column=1, padx=10, pady=5, sticky="ew")

label_prontuario = tk.Label(janela, text="Prontuário (1-20300):", font=bold_font, anchor="w", bg="#ADD8E6")
label_prontuario.grid(row=1, column=0, sticky="w", padx=10, pady=5)
entrada_prontuario = tk.Entry(janela, font=("Arial", 12), width=30)
entrada_prontuario.grid(row=1, column=1, padx=10, pady=5, sticky="ew")

label_endereco = tk.Label(janela, text="Endereço:", font=bold_font, anchor="w", bg="#ADD8E6")
label_endereco.grid(row=2, column=0, sticky="w", padx=10, pady=5)
entrada_endereco = tk.Entry(janela, font=("Arial", 12), width=30)
entrada_endereco.grid(row=2, column=1, padx=10, pady=5, sticky="ew")

label_search = tk.Label(janela, text="Pesquisar:", font=bold_font, anchor="w", bg="#ADD8E6")
label_search.grid(row=3, column=0, sticky="w", padx=10, pady=5)
entrada_search = tk.Entry(janela, font=("Arial", 12), width=30)
entrada_search.grid(row=3, column=1, padx=10, pady=5, sticky="ew")

entrada_nome.bind("<Return>", add)
entrada_prontuario.bind("<Return>", add)
entrada_endereco.bind("<Return>", add)

# Botões ajustados
button_style = {"font": ("Arial", 10, "bold"), "width": 20, "bg": "#5F9EA0", "fg": "white"}
button_frame = tk.Frame(janela, bg="#ADD8E6")
button_frame.grid(row=4, column=0, rowspan=3, padx=10, pady=5, sticky="nsew")

add_button = tk.Button(button_frame, text="Adicionar", command=add, **button_style)
add_button.pack(fill="x", pady=2)

search_button = tk.Button(button_frame, text="Pesquisar", command=search, **button_style)
search_button.pack(fill="x", pady=2)

delete_button = tk.Button(button_frame, text="Deletar", command=delete, **button_style)
delete_button.pack(fill="x", pady=2)

import_button = tk.Button(button_frame, text="Importar do Excel", command=import_from_excel, **button_style)
import_button.pack(fill="x", pady=2)

# Tabela
tree = ttk.Treeview(janela, columns=('nome', 'prontuario', 'endereco'), show='headings', height=10, selectmode="browse")
tree.heading('nome', text='Nome', anchor="center")
tree.heading('prontuario', text='Prontuário', anchor="center")
tree.heading('endereco', text='Endereço', anchor="w")  # Endereço alinhado à esquerda

tree.column('nome', anchor="center")
tree.column('prontuario', anchor="center")
tree.column('endereco', anchor="w")  # Endereço alinhado à esquerda

tree.grid(row=4, column=1, rowspan=3, padx=10, pady=5, sticky="nsew")

scrollbar = ttk.Scrollbar(janela, orient="vertical", command=tree.yview)
tree.configure(yscroll=scrollbar.set)
scrollbar.grid(row=4, column=2, rowspan=3, sticky="ns")

style = ttk.Style(janela)
style.configure("Treeview.Heading", font=("Arial", 10, "bold"), relief="solid")
style.configure("Treeview", rowheight=25, borderwidth=1, relief="solid")
style.map("Treeview", background=[("selected", "#347083")])

update_table()

janela.mainloop()
conn.close()
