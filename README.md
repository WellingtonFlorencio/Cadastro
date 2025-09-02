import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
from datetime import datetime


class AcademiaApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Gestão de Alunos - Academia")
        self.root.geometry("800x650")

        # Cores e fontes para o tema de academia
        self.bg_color = "#2c3e50"
        self.fg_color = "#ecf0f1"
        self.accent_color = "#e74c3c"
        self.font_large = ("Helvetica", 12, "bold")
        self.font_small = ("Helvetica", 10)

        self.root.configure(bg=self.bg_color)

        self.inicializar_banco()
        self.criar_interface()
        self.carregar_clientes()

    def inicializar_banco(self):
        """Conecta ao banco de dados e cria a tabela de alunos se ela não existir."""
        self.conn = sqlite3.connect("academia.db")
        self.cursor = self.conn.cursor()
        self.cursor.execute("""
                            CREATE TABLE IF NOT EXISTS clientes
                            (
                                id
                                INTEGER
                                PRIMARY
                                KEY
                                AUTOINCREMENT,
                                nome
                                TEXT
                                NOT
                                NULL,
                                idade
                                INTEGER,
                                telefone
                                TEXT,
                                email
                                TEXT,
                                plano
                                TEXT,
                                data_adesao
                                TEXT
                            );
                            """)
        self.conn.commit()

    def criar_interface(self):
        """Cria os widgets da interface gráfica com o tema."""
        style = ttk.Style()
        style.theme_use("clam")
        style.configure("TFrame", background=self.bg_color)
        style.configure("TLabel", background=self.bg_color, foreground=self.fg_color, font=self.font_large)
        style.configure("TButton", background=self.accent_color, foreground="white", font=self.font_small)
        style.map("TButton", background=[("active", "#c0392b")])
        style.configure("Treeview", background=self.fg_color, foreground="black", font=self.font_small)
        style.configure("Treeview.Heading", font=self.font_large, background=self.accent_color, foreground="white")

        # Frame de Entrada de Dados
        frame_input = ttk.LabelFrame(self.root, text="Dados do Aluno", style="TFrame")
        frame_input.pack(pady=10, padx=10, fill="x")

        # Usando um dicionário literal para maior clareza e controle das chaves
        self.entries = {}

        ttk.Label(frame_input, text="Nome:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.entries["nome"] = ttk.Entry(frame_input, width=40)
        self.entries["nome"].grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(frame_input, text="Idade:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.entries["idade"] = ttk.Entry(frame_input, width=40)
        self.entries["idade"].grid(row=1, column=1, padx=5, pady=5)

        ttk.Label(frame_input, text="Telefone:").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.entries["telefone"] = ttk.Entry(frame_input, width=40)
        self.entries["telefone"].grid(row=2, column=1, padx=5, pady=5)

        ttk.Label(frame_input, text="Email:").grid(row=3, column=0, padx=5, pady=5, sticky="w")
        self.entries["email"] = ttk.Entry(frame_input, width=40)
        self.entries["email"].grid(row=3, column=1, padx=5, pady=5)

        ttk.Label(frame_input, text="Data da Adesão:").grid(row=4, column=0, padx=5, pady=5, sticky="w")
        self.entries["data_adesao"] = ttk.Entry(frame_input, width=40)
        self.entries["data_adesao"].grid(row=4, column=1, padx=5, pady=5)
        self.entries["data_adesao"].insert(0, datetime.now().strftime("%d/%m/%Y"))

        # Campo de Plano (Combobox)
        ttk.Label(frame_input, text="Plano:").grid(row=5, column=0, padx=5, pady=5, sticky="w")
        self.combo_plano = ttk.Combobox(frame_input, values=["Bronze", "Prata", "Ouro"], state="readonly")
        self.combo_plano.grid(row=5, column=1, padx=5, pady=5)
        self.combo_plano.set("Bronze")  # Valor padrão

        # Frame de Botões
        frame_botoes = ttk.Frame(self.root, style="TFrame")
        frame_botoes.pack(pady=5)

        ttk.Button(frame_botoes, text="Adicionar", command=self.adicionar_cliente).pack(side="left", padx=5)
        ttk.Button(frame_botoes, text="Atualizar Selecionado", command=self.atualizar_cliente).pack(side="left", padx=5)
        ttk.Button(frame_botoes, text="Remover", command=self.remover_cliente).pack(side="left", padx=5)

        # Treeview para exibir a lista de clientes
        self.tree = ttk.Treeview(self.root,
                                 columns=("ID", "Nome", "Idade", "Telefone", "Email", "Plano", "Data da Adesão"),
                                 show="headings")
        self.tree.heading("ID", text="ID")
        self.tree.heading("Nome", text="Nome")
        self.tree.heading("Idade", text="Idade")
        self.tree.heading("Telefone", text="Telefone")
        self.tree.heading("Email", text="Email")
        self.tree.heading("Plano", text="Plano")
        self.tree.heading("Data da Adesão", text="Data da Adesão")
        self.tree.column("ID", width=40, anchor="center")
        self.tree.column("Nome", width=120)
        self.tree.column("Idade", width=60, anchor="center")
        self.tree.column("Telefone", width=100)
        self.tree.column("Email", width=150)
        self.tree.column("Plano", width=80)
        self.tree.column("Data da Adesão", width=100)
        self.tree.pack(fill="both", expand=True, padx=10, pady=10)

        self.tree.bind("<<TreeviewSelect>>", self.carregar_campos)

    def carregar_clientes(self):
        """Carrega e exibe todos os clientes na Treeview."""
        for item in self.tree.get_children():
            self.tree.delete(item)

        self.cursor.execute("SELECT id, nome, idade, telefone, email, plano, data_adesao FROM clientes")
        clientes = self.cursor.fetchall()
        for cliente in clientes:
            self.tree.insert("", "end", values=cliente)

    def carregar_campos(self, event):
        """Preenche os campos de entrada com os dados do cliente selecionado."""
        item_selecionado = self.tree.selection()
        if item_selecionado:
            values = self.tree.item(item_selecionado, "values")
            self.limpar_campos()
            self.entries["nome"].insert(0, values[1])
            self.entries["idade"].insert(0, values[2])
            self.entries["telefone"].insert(0, values[3])
            self.entries["email"].insert(0, values[4])
            self.combo_plano.set(values[5])
            self.entries["data_adesao"].insert(0, values[6])

    def adicionar_cliente(self):
        """Adiciona um cliente ao banco de dados e atualiza a interface."""
        nome = self.entries["nome"].get()
        try:
            idade = int(self.entries["idade"].get())
        except (ValueError, IndexError):
            messagebox.showerror("Erro", "Idade deve ser um número inteiro.")
            return
        telefone = self.entries["telefone"].get()
        email = self.entries["email"].get()
        plano = self.combo_plano.get()
        data_adesao = self.entries["data_adesao"].get()

        if not nome or not telefone or not email or not plano or not data_adesao:
            messagebox.showerror("Erro", "Todos os campos são obrigatórios.")
            return

        self.cursor.execute(
            "INSERT INTO clientes (nome, idade, telefone, email, plano, data_adesao) VALUES (?, ?, ?, ?, ?, ?)",
            (nome, idade, telefone, email, plano, data_adesao))
        self.conn.commit()
        self.carregar_clientes()
        self.limpar_campos()
        messagebox.showinfo("Sucesso", "Aluno adicionado com sucesso!")

    def remover_cliente(self):
        """Remove o cliente selecionado na Treeview."""
        item_selecionado = self.tree.selection()
        if not item_selecionado:
            messagebox.showwarning("Aviso", "Selecione um aluno para remover.")
            return

        id_cliente = self.tree.item(item_selecionado, "values")[0]

        if messagebox.askyesno("Confirmação", f"Tem certeza que deseja remover o aluno de ID {id_cliente}?"):
            self.cursor.execute("DELETE FROM clientes WHERE id = ?", (id_cliente,))
            self.conn.commit()
            self.carregar_clientes()
            self.limpar_campos()
            messagebox.showinfo("Sucesso", "Aluno removido com sucesso!")

    def atualizar_cliente(self):
        """Atualiza os dados do cliente selecionado."""
        item_selecionado = self.tree.selection()
        if not item_selecionado:
            messagebox.showwarning("Aviso", "Selecione um aluno para atualizar.")
            return

        id_cliente = self.tree.item(item_selecionado, "values")[0]
        nome = self.entries["nome"].get()
        try:
            idade = int(self.entries["idade"].get())
        except (ValueError, IndexError):
            messagebox.showerror("Erro", "Idade deve ser um número inteiro.")
            return
        telefone = self.entries["telefone"].get()
        email = self.entries["email"].get()
        plano = self.combo_plano.get()
        data_adesao = self.entries["data_adesao"].get()

        if not nome or not telefone or not email or not plano or not data_adesao:
            messagebox.showerror("Erro", "Todos os campos são obrigatórios para a atualização.")
            return

        self.cursor.execute("""
                            UPDATE clientes
                            SET nome        = ?,
                                idade       = ?,
                                telefone    = ?,
                                email       = ?,
                                plano       = ?,
                                data_adesao = ?
                            WHERE id = ?
                            """, (nome, idade, telefone, email, plano, data_adesao, id_cliente))

        self.conn.commit()
        self.carregar_clientes()
        self.limpar_campos()
        messagebox.showinfo("Sucesso", "Dados do aluno atualizados!")

    def limpar_campos(self):
        """Limpa as caixas de entrada de texto e o Combobox."""
        for key in self.entries:
            self.entries[key].delete(0, tk.END)
        self.combo_plano.set("Bronze")


if __name__ == "__main__":
    root = tk.Tk()
    app = AcademiaApp(root)
    root.mainloop()
