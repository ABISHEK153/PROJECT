# PROJECT
import tkinter as tk
from tkinter import ttk, messagebox, filedialog
import pandas as pd
from reportlab.lib import colors
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet
import random

root = tk.Tk()
root.title("Grocery Billing Form - WORLD BASKET")
root.geometry("850x600")

billing_data = []

form_frame = tk.Frame(root)
form_frame.pack(pady=10)

tk.Label(form_frame, text="Item Name:").grid(row=0, column=0, padx=10, pady=5)
item_entry = tk.Entry(form_frame)
item_entry.grid(row=0, column=1, padx=10, pady=5)

tk.Label(form_frame, text="Quantity:").grid(row=1, column=0, padx=10, pady=5)
qty_entry = tk.Entry(form_frame)
qty_entry.grid(row=1, column=1, padx=10, pady=5)

tk.Label(form_frame, text="Price per Item (Rs):").grid(row=2, column=0, padx=10, pady=5)
price_entry = tk.Entry(form_frame)
price_entry.grid(row=2, column=1, padx=10, pady=5)

tree_frame = tk.Frame(root)
tree_frame.pack(pady=10)

columns = ("Item Name", "Quantity", "Price per Item (Rs)", "Total (Rs)")
tree = ttk.Treeview(tree_frame, columns=columns, show="headings")

for col in columns:
    tree.heading(col, text=col)
    tree.column(col, width=160)
tree.pack()

total_label = tk.Label(root, text="Total Bill: Rs0.00", font=("Arial", 12, "bold"))
total_label.pack(pady=10)

def add_item():
    item = item_entry.get()
    qty = qty_entry.get()
    price = price_entry.get()
    if not item or not qty or not price:
        messagebox.showerror("Missing Data", "Please fill all fields!")
        return
    try:
        qty = float(qty)
        price = float(price)
        total = qty * price
    except ValueError:
        messagebox.showerror("Invalid Input", "Enter valid numbers for Quantity and Price.")
        return
    billing_data.append([item, qty, price, total])
    tree.insert("", "end", values=(item, qty, f"Rs{price:.2f}", f"Rs{total:.2f}"))
    update_total()
    item_entry.delete(0, tk.END)
    qty_entry.delete(0, tk.END)
    price_entry.delete(0, tk.END)

def update_total():
    if not billing_data:
        total_label.config(text="Total Bill: Rs0.00")
        return
    df = pd.DataFrame(billing_data, columns=columns)
    total_bill = df["Total (Rs)"].sum()
    cgst = total_bill * 0.08
    sgst = total_bill * 0.08
    final_total = total_bill + cgst + sgst
    total_label.config(
        text=f"Subtotal: Rs{total_bill:.2f} | CGST (8%): Rs{cgst:.2f} | SGST (8%): Rs{sgst:.2f} | Final Total: Rs{final_total:.2f}"
    )

def export_csv():
    if not billing_data:
        messagebox.showinfo("No Data", "No billing data to export.")
        return
    df = pd.DataFrame(billing_data, columns=columns)
    file_path = filedialog.asksaveasfilename(defaultextension=".csv",
                                             filetypes=[("CSV Files", "*.csv")])
    if file_path:
        df.to_csv(file_path, index=False)
        messagebox.showinfo("Export Successful", f"Data exported to {file_path}")

def export_pdf():
    if not billing_data:
        messagebox.showinfo("No Data", "No billing data to export.")
        return
    file_path = filedialog.asksaveasfilename(defaultextension=".pdf",
                                             filetypes=[("PDF Files", "*.pdf")])
    if not file_path:
        return
    df = pd.DataFrame(billing_data, columns=columns)
    subtotal = df["Total (Rs)"].sum()
    cgst = subtotal * 0.08
    sgst = subtotal * 0.08
    final_total = subtotal + cgst + sgst
    invoice_no = random.randint(1000000000, 9999999999)
    pdf = SimpleDocTemplate(file_path, pagesize=A4)
    elements = []
    styles = getSampleStyleSheet()
    elements += [
        Paragraph("<b>WORLD BASKET</b>", styles["Title"]),
        Paragraph("<b>TAX INVOICE</b>", styles["Heading2"]),
        Paragraph(f"Invoice No: {invoice_no}", styles["Normal"]),
        Spacer(1, 15),
    ]
    table_data = [list(df.columns)] + df.values.tolist()
    table_data += [
        ["", "", "Subtotal", f"Rs {subtotal:.2f}"],
        ["", "", "CGST (8%)", f"Rs {cgst:.2f}"],
        ["", "", "SGST (8%)", f"Rs {sgst:.2f}"],
        ["", "", "Grand Total", f"Rs {final_total:.2f}"],
    ]
    table = Table(table_data, hAlign="CENTER")
    table.setStyle(TableStyle([
        ("BACKGROUND", (0, 0), (-1, 0), colors.lightblue),
        ("ALIGN", (0, 0), (-1, -1), "CENTER"),
        ("GRID", (0, 0), (-1, -1), 1, colors.black),
        ("FONTNAME", (0, 0), (-1, 0), "Helvetica-Bold"),
    ]))
    elements.append(table)
    elements.append(Spacer(1, 20))
    elements.append(Paragraph("Thank you for shopping at WORLD BASKET!", styles["Normal"]))
    pdf.build(elements)
    messagebox.showinfo("PDF Exported", f"PDF invoice saved successfully at {file_path}")

button_frame = tk.Frame(root)
button_frame.pack(pady=10)
tk.Button(button_frame, text="Add Item", command=add_item, width=15).grid(row=0, column=0, padx=10)
tk.Button(button_frame, text="Export to CSV", command=export_csv, width=15).grid(row=0, column=1, padx=10)
tk.Button(button_frame, text="Export to PDF", command=export_pdf, width=15).grid(row=0, column=2, padx=10)
root.mainloop()
