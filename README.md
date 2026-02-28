# data-gri
import PyPDF2
import os
import pandas as pd

# Masukkan folder tempat Anda menyimpan file PDF Laporan Keberlanjutan
pdf_folder = "jalur/ke/folder/sustainability_reports"
gri_indicators = ["GRI 301", "GRI 302", "GRI 303", "GRI 304", "GRI 305"]
results = []

for filename in os.listdir(pdf_folder):
    if filename.endswith(".pdf"):
        filepath = os.path.join(pdf_folder, filename)
        
        # Inisialisasi status pengungkapan (0 = tidak ada, 1 = ada)
        disclosure = {ind: 0 for ind in gri_indicators}
        
        try:
            with open(filepath, "rb") as file:
                reader = PyPDF2.PdfReader(file)
                # Ekstrak teks dari beberapa halaman terakhir (biasanya lokasi Indeks GRI)
                num_pages = len(reader.pages)
                start_page = max(0, num_pages - 30) # Cek 30 halaman terakhir
                
                text = ""
                for page_num in range(start_page, num_pages):
                    page = reader.pages[page_num]
                    text += page.extract_text() or ""
                
                # Cek keberadaan indikator GRI
                for ind in gri_indicators:
                    if ind in text:
                        disclosure[ind] = 1
                        
            # Simpan hasil untuk file ini
            total_score = sum(disclosure.values())
            results.append({
                "File": filename,
                "GRI 301": disclosure["GRI 301"],
                "GRI 302": disclosure["GRI 302"],
                "GRI 303": disclosure["GRI 303"],
                "GRI 304": disclosure["GRI 304"],
                "GRI 305": disclosure["GRI 305"],
                "Total Pengungkapan": total_score
            })
        except Exception as e:
            print(f"Gagal membaca {filename}: {e}")

# Buat dataframe dan simpan ke Excel
df = pd.DataFrame(results)
df.to_excel("Hasil_Ekstraksi_GRI.xlsx", index=False)
print("Ekstraksi selesai, file disimpan sebagai Hasil_Ekstraksi_GRI.xlsx")
