# controle-financeiro
App de controle de gastos

from flask import Flask, request
import sqlite3
from datetime import datetime

app = Flask(__name__)

def criar_banco():
    conn = sqlite3.connect("gastos.db")
    c = conn.cursor()
    c.execute("""
    CREATE TABLE IF NOT EXISTS gastos (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        valor REAL,
        categoria TEXT,
        data TEXT
    )
    """)
    conn.commit()
    conn.close()

def salvar(valor, categoria):
    conn = sqlite3.connect("gastos.db")
    c = conn.cursor()
    c.execute("INSERT INTO gastos (valor, categoria, data) VALUES (?, ?, ?)",
              (valor, categoria, datetime.now().strftime("%Y-%m-%d")))
    conn.commit()
    conn.close()

def relatorio():
    conn = sqlite3.connect("gastos.db")
    c = conn.cursor()

    total = sum([row[0] for row in c.execute("SELECT valor FROM gastos")])

    texto = f"📊 RELATÓRIO\nTotal: R$ {total:.2f}\n\n"

    for row in c.execute("SELECT categoria, SUM(valor) FROM gastos GROUP BY categoria"):
        texto += f"{row[0]}: R$ {row[1]:.2f}\n"

    conn.close()
    return texto

@app.route("/webhook", methods=["POST"])
def webhook():
    msg = request.form.get("Body").lower()

    if "relatorio" in msg:
        return relatorio()

    try:
        partes = msg.split()
        valor = float([p for p in partes if p.replace('.', '', 1).isdigit()][0])
        categoria = partes[-1]
        salvar(valor, categoria)
        return f"✅ Salvo R$ {valor} em {categoria}"
    except:
        return "❌ Exemplo: 50 mercado ou relatorio"

if __name__ == "__main__":
    criar_banco()
    app.run()
