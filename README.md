import tkinter as tk
from tkinter import ttk, simpledialog, messagebox
import pyodbc

def get_connection():
    try:
        conn = pyodbc.connect(
            "DRIVER={ODBC Driver 17 for SQL Server};"
            "SERVER=DESKTOP-8FRGTHL\\SQLEXPRESS,1433;"
            "DATABASE=EstudiantesDB;"
            "UID=curso2c;"
            "PWD=curso2c;"
        )
        return conn
    except Exception as e:
        messagebox.showerror("Error de conexión", f"No se pudo conectar a SQL Server:\n{e}")
        return None
    
class QueryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Consultas con Botones - SQL Server")

        self.queries = {
            "Ver Estudiantes": "SELECT * FROM estudiantes;",
        }

        self.frame_buttons = tk.Frame(self.root)
        self.frame_buttons.pack(side=tk.LEFT, fill=tk.Y, padx=5, pady=5)

      
        tk.Button(self.frame_buttons, text="➕ Nueva consulta", command=self.add_query).pack(fill="x", pady=2)
        tk.Button(self.frame_buttons, text="✏️ Editar consulta", command=self.edit_query).pack(fill="x", pady=2)
        tk.Button(self.frame_buttons, text="🗑️ Eliminar consulta", command=self.delete_query).pack(fill="x", pady=2)

      
        self.frame_table = tk.Frame(self.root)
        self.frame_table.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)

    
        self.table = ttk.Treeview(self.frame_table)
        self.table.pack(fill=tk.BOTH, expand=True)

    
        self.load_buttons()

    def load_buttons(self):
    
        for widget in self.frame_buttons.winfo_children():
            if isinstance(widget, tk.Button) and widget.cget("text") not in ["➕ Nueva consulta", "✏️ Editar consulta", "🗑️ Eliminar consulta"]:
                widget.destroy()

        for name, query in self.queries.items():
            b = tk.Button(self.frame_buttons, text=name, command=lambda q=query: self.run_query(q))
            b.pack(fill="x", pady=2)

    def run_query(self, query):
        conn = get_connection()
        if conn is None:
            return
        try:
            cursor = conn.cursor()
            cursor.execute(query)
            rows = cursor.fetchall()
            columns = [desc[0] for desc in cursor.description]

            # Reiniciar tabla
            self.table.delete(*self.table.get_children())
            self.table["columns"] = columns
            self.table["show"] = "headings"

            for col in columns:
                self.table.heading(col, text=col)
                self.table.column(col, width=120)

            for row in rows:
                self.table.insert("", "end", values=row)

            conn.close()
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo ejecutar la consulta:\n{e}")

    def add_query(self):
        name = simpledialog.askstring("Nueva consulta", "Nombre del botón:")
        if not name:
            return
        query = simpledialog.askstring("Nueva consulta", "Escribe la consulta SQL:")
        if not query:
            return
        self.queries[name] = query
        self.load_buttons()

    def edit_query(self):
        if not self.queries:
            messagebox.showinfo("Editar", "No hay consultas para editar.")
            return
        name = simpledialog.askstring("Editar consulta", f"Consultas disponibles:\n{list(self.queries.keys())}\n\nEscribe el nombre del botón a editar:")
        if not name or name not in self.queries:
            return
        new_query = simpledialog.askstring("Editar consulta", "Escribe la nueva consulta SQL:", initialvalue=self.queries[name])
        if new_query:
            self.queries[name] = new_query
            self.load_buttons()

    def delete_query(self):
        if not self.queries:
            messagebox.showinfo("Eliminar", "No hay consultas para eliminar.")
            return
        name = simpledialog.askstring("Eliminar consulta", f"Consultas disponibles:\n{list(self.queries.keys())}\n\nEscribe el nombre del botón a eliminar:")
        if not name or name not in self.queries:
            return
        confirm = messagebox.askyesno("Confirmar", f"¿Seguro que deseas eliminar la consulta '{name}'?")
        if confirm:
            del self.queries[name]
            self.load_buttons()

if __name__ == "__main__":
    root = tk.Tk()
    app = QueryApp(root)
    root.mainloop()
