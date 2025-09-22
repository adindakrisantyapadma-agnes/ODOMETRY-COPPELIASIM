# ODOMETRY-COPPELIASIM
# Adinda Krisantya 5022221156



import matplotlib.pyplot as plt
import numpy as np
import math
import time
from fpdf import FPDF
from datetime import datetime
import os

class P3DXSimulator:
    def __init__(self):
        print("Mode Simulasi P3DX - Menggunakan data simulasi")
    
    def generate_simulation_data(self, simulation_time=30, sampling_rate=10):
        """Generate sample data untuk simulasi P3DX"""
        t = np.linspace(0, simulation_time, simulation_time * sampling_rate)
        
        # Trajektori kompleks untuk simulasi yang lebih realistis
        # Kombinasi garis lurus dan kurva
        x_data = []
        y_data = []
        yaw_data = []
        
        for time_val in t:
            if time_val < 10:
                # Fase 1: Gerak lurus
                x = 0.5 * time_val
                y = 0.2 * np.sin(0.5 * time_val)
                yaw = 5 * time_val
            elif time_val < 20:
                # Fase 2: Belok kanan
                theta = 0.3 * (time_val - 10)
                x = 5 + 3 * np.cos(theta)
                y = 2 + 3 * np.sin(theta)
                yaw = 50 + math.degrees(theta)
            else:
                # Fase 3: Gerak lurus kembali
                x = 5 + 3 * np.cos(3) - 0.5 * (time_val - 20)
                y = 2 + 3 * np.sin(3)
                yaw = 50 + math.degrees(3)
            
            x_data.append(x)
            y_data.append(y)
            yaw_data.append(yaw)
        
        return t, x_data, y_data, yaw_data
    
    def create_temporal_plots(self, time_data, x_data, y_data, yaw_data):
        """Buat plot temporal"""
        plt.figure(figsize=(15, 10))
        
        # Plot posisi x vs waktu
        plt.subplot(3, 1, 1)
        plt.plot(time_data, x_data, 'b-', linewidth=2)
        plt.xlabel('Waktu (detik)')
        plt.ylabel('Posisi X (m)')
        plt.title('Plot Temporal Posisi X vs Waktu - P3DX')
        plt.grid(True, alpha=0.3)
        
        # Plot posisi y vs waktu
        plt.subplot(3, 1, 2)
        plt.plot(time_data, y_data, 'r-', linewidth=2)
        plt.xlabel('Waktu (detik)')
        plt.ylabel('Posisi Y (m)')
        plt.title('Plot Temporal Posisi Y vs Waktu - P3DX')
        plt.grid(True, alpha=0.3)
        
        # Plot yaw vs waktu
        plt.subplot(3, 1, 3)
        plt.plot(time_data, yaw_data, 'g-', linewidth=2)
        plt.xlabel('Waktu (detik)')
        plt.ylabel('Yaw (derajat)')
        plt.title('Plot Temporal Yaw vs Waktu - P3DX')
        plt.grid(True, alpha=0.3)
        
        plt.tight_layout()
        plt.savefig('temporal_plot_p3dx.png', dpi=300, bbox_inches='tight')
        plt.show()
    
    def create_spatial_plot(self, x_data, y_data):
        """Buat plot spasial"""
        plt.figure(figsize=(10, 8))
        plt.plot(x_data, y_data, 'b-', linewidth=2, label='Trajektori P3DX', alpha=0.7)
        
        # Tambah titik awal dan akhir
        if len(x_data) > 0:
            plt.plot(x_data[0], y_data[0], 'go', markersize=10, label='Start Point', markeredgecolor='black')
            plt.plot(x_data[-1], y_data[-1], 'ro', markersize=10, label='End Point', markeredgecolor='black')
            
            # Tambah panah untuk menunjukkan arah
            mid_idx = len(x_data) // 2
            plt.annotate('', xy=(x_data[mid_idx+1], y_data[mid_idx+1]), 
                        xytext=(x_data[mid_idx], y_data[mid_idx]),
                        arrowprops=dict(arrowstyle='->', color='red', lw=2))
        
        plt.xlabel('Posisi X (m)')
        plt.ylabel('Posisi Y (m)')
        plt.title('Plot Spasial Trajektori P3DX')
        plt.legend()
        plt.grid(True, alpha=0.3)
        plt.axis('equal')
        
        plt.savefig('spatial_plot_p3dx.png', dpi=300, bbox_inches='tight')
        plt.show()
    
    def save_data_to_csv(self, time_data, x_data, y_data, yaw_data):
        """Simpan data ke file CSV"""
        data = np.column_stack((time_data, x_data, y_data, yaw_data))
        np.savetxt('p3dx_pose_data.csv', data, delimiter=',', 
                   header='Time(s),X(m),Y(m),Yaw(deg)', comments='', fmt='%.4f')
        print('Data disimpan sebagai: p3dx_pose_data.csv')

class PDFReport(FPDF):
    def header(self):
        self.set_font('Arial', 'B', 12)
        self.cell(0, 10, 'Laporan Analisis Pose Robot P3DX', 0, 1, 'C')
        self.ln(5)
    
    def footer(self):
        self.set_y(-15)
        self.set_font('Arial', 'I', 8)
        self.cell(0, 10, f'Halaman {self.page_no()}', 0, 0, 'C')
    
    def add_plot(self, image_path, title, width=180):
        self.ln(10)
        self.set_font('Arial', 'B', 12)
        self.cell(0, 10, title, 0, 1, 'L')
        if os.path.exists(image_path):
            self.image(image_path, x=15, y=None, w=width)
            self.ln(5)
        else:
            self.cell(0, 10, f'Gambar {image_path} tidak ditemukan', 0, 1)

def create_pdf_report():
    """Buat laporan PDF"""
    pdf = PDFReport()
    pdf.add_page()
    
    # Judul
    pdf.set_font('Arial', 'B', 16)
    pdf.cell(0, 10, 'LAPORAN ANALISIS POSE ROBOT P3DX', 0, 1, 'C')
    pdf.ln(10)
    
    # Informasi laporan
    pdf.set_font('Arial', '', 12)
    pdf.cell(0, 10, f'Tanggal: {datetime.now().strftime("%Y-%m-%d %H:%M:%S")}', 0, 1)
    pdf.cell(0, 10, 'Robot: Pioneer P3DX', 0, 1)
    pdf.cell(0, 10, 'Mode: Simulasi Data', 0, 1)
    pdf.ln(10)
    
    # Deskripsi
    pdf.multi_cell(0, 10, 'Laporan ini berisi analisis pose robot P3DX yang meliputi plot temporal dan spasial dari pergerakan robot selama simulasi.')
    pdf.ln(10)
    
    # Tambah plot temporal
    pdf.add_plot('temporal_plot_p3dx.png', 'Grafik Temporal Pose P3DX')
    
    pdf.add_page()
    
    # Tambah plot spasial
    pdf.add_plot('spatial_plot_p3dx.png', 'Grafik Spasial Trajektori P3DX')
    
    # Analisis data
    pdf.add_page()
    pdf.set_font('Arial', 'B', 14)
    pdf.cell(0, 10, 'Analisis Data', 0, 1)
    pdf.ln(5)
    
    try:
        data = np.genfromtxt('p3dx_pose_data.csv', delimiter=',', skip_header=1)
        if data.size > 0:
            pdf.set_font('Arial', '', 12)
            pdf.cell(0, 10, f'Jumlah sampel data: {len(data)}', 0, 1)
            pdf.cell(0, 10, f'Durasi pengamatan: {data[-1,0]:.2f} detik', 0, 1)
            
            # Hitung jarak tempuh
            distances = np.sqrt(np.diff(data[:,1])**2 + np.diff(data[:,2])**2)
            total_distance = np.sum(distances)
            pdf.cell(0, 10, f'Jarak tempuh total: {total_distance:.2f} m', 0, 1)
            
            # Perpindahan
            displacement = np.sqrt((data[-1,1]-data[0,1])**2 + (data[-1,2]-data[0,2])**2)
            pdf.cell(0, 10, f'Perpindahan: {displacement:.2f} m', 0, 1)
            
            # Yaw awal dan akhir
            pdf.cell(0, 10, f'Yaw awal: {data[0,3]:.1f}°', 0, 1)
            pdf.cell(0, 10, f'Yaw akhir: {data[-1,3]:.1f}°', 0, 1)
    except Exception as e:
        pdf.cell(0, 10, f'Error dalam analisis data: {e}', 0, 1)
    
    # Simpan PDF
    pdf.output('p3dx_pose_report.pdf')
    print('Laporan PDF telah disimpan sebagai: p3dx_pose_report.pdf')

def main():
    """Fungsi utama"""
    # Inisialisasi simulator
    simulator = P3DXSimulator()
    
    # Generate data simulasi
    print("Generating simulation data...")
    time_data, x_data, y_data, yaw_data = simulator.generate_simulation_data(30, 10)
    
    # Buat plot
    print("Creating temporal plots...")
    simulator.create_temporal_plots(time_data, x_data, y_data, yaw_data)
    
    print("Creating spatial plot...")
    simulator.create_spatial_plot(x_data, y_data)
    
    # Simpan data
    print("Saving data to CSV...")
    simulator.save_data_to_csv(time_data, x_data, y_data, yaw_data)
    
    # Buat laporan PDF
    print("Generating PDF report...")
    create_pdf_report()
    
    print("\n=== SIMULASI SELESAI ===")
    print("File yang dihasilkan:")
    print("1. temporal_plot_p3dx.png - Plot temporal")
    print("2. spatial_plot_p3dx.png - Plot spasial") 
    print("3. p3dx_pose_data.csv - Data mentah")
    print("4. p3dx_pose_report.pdf - Laporan lengkap")

if __name__ == "__main__":
    main()
