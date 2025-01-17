import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Configure the page layout
st.set_page_config(page_title="Antioxidant Calc")

image_url = "https://png.pngtree.com/background/20230525/original/pngtree-an-image-of-ocean-at-night-with-dark-clouds-picture-image_2734744.jpg"

# CSS untuk mengatur gambar sebagai background
st.markdown(
    f"""
    <style>
    .stApp {{
        background-image: url('{image_url}');
        background-size: cover;
        background-position: center;
        height: 100vh;
    }}
    </style>
    """, 
    unsafe_allow_html=True
)

# Initialize session state if not already done
if "current_page" not in st.session_state:
    st.session_state.current_page = "Home"

# Sidebar Navigation
def sidebar():
    pages = ["Home", "About Us", "Contact"]
    icons = ["🏠", "👤", "☎"]  # Icons for each page
    page = st.sidebar.selectbox("Navigate", [f"{icons[i]} {page}" for i, page in enumerate(pages)])
    st.session_state.current_page = pages[[f"{icons[i]} {page}" for i, page in enumerate(pages)].index(page)]
    return st.session_state.current_page

def home():
    st.markdown("""<style>.title {color: pink; text-align: center; font-size: 40px;} .custom-button {background-color: #32CD32; color: white; border: 1px solid white; border-radius: 10px; padding: 10px 20px; cursor: pointer; display: inline-block;} .custom-button:hover {background-color: #28a745;} .center-content {text-align: center;}</style>""", unsafe_allow_html=True)
    st.markdown("<h1 class='title'>Antioxidant Calc</h1>", unsafe_allow_html=True)

    if st.button("Start to Calculate", key="start_button", use_container_width=True):
        st.session_state.current_page = "Page 2"

    st.write("""
    <div style="text-align: center; margin-top: 20px;">
        Antioksidan adalah suatu zat yang dapat melindungi senyawa kimia dalam tubuh dari oksidasi yang dapat merusak dengan cara bereaksi dengan radikal bebas dan spesi oksigen reaktif, sehingga dapat menghambat oksidasi. Antioksidan juga disebut sebagai scavenger (zat/peredam) radikal bebas dan dapat menetralkan radikal bebas.<br><br>
        Antioksidan sebagai senyawa yang dapat menonaktifkan radikal bebas dengan menggunakan dua mekanisme utama yaitu Hidrogen Atom Transfer (HAT) dan Single Electron Transfer (SET). Kedua mekanisme tersebut menjadi dasar metode pengujian antioksidan.<br><br>
        Metode HAT mengukur kemampuan antioksidan untuk meredam radikal bebas dengan sumbangan hidrogen (donor proton), Metode SET mengukur kemampuan antioksidan untuk mereduksi radikal bebas melalui transfer elektron tunggal. Sedangkan pada mekanisme SET, antioksidan mendonasikan satu elektron ke radikal bebas sehingga radikal bebas menjadi netral dan stabil. Proses ini mencegah radikal bebas merusak molekul biologis seperti lipid, protein, dan DNA.
    </div>
    """, unsafe_allow_html=True)

def page_2():
    st.title("Select a Test Method")

    test_methods = ["ORAC", "TRAP", "LPIC", "FRAP", "TEAC", "DPPH", "CUPRAC", "ABTS"]
    cols = st.columns(3)
    for i, method in enumerate(test_methods):
        with cols[i % 3]:
            if st.button(method, key=method):
                st.session_state.current_page = f"ATOX.CALC-{method}"

def atox_calc_page(method):
    if method == "DPPH":
        st.markdown("<h1 style='color: violet; text-align: center;'>DPPH</h1>", unsafe_allow_html=True)
        blank = st.number_input("Input blanko", format="%.6f")
        num_samples = st.number_input("Berapa banyak sampel ingin diuji", min_value=0, step=1)

        if num_samples > 0:
            st.write("Masukkan konsentrasi (ppm) dan absorbansi untuk setiap sampel")
            cols = st.columns(2)
            concentrations = []
            absorbances = []
            for i in range(num_samples):
                with cols[0]:
                    concentrations.append(st.number_input(f"Input konsentrasi (ppm) sampel {i+1}", format="%.6f"))
                with cols[1]:
                    absorbances.append(st.number_input(f"Input absorbansi sampel {i+1}", format="%.6f"))

            # Calculate % inhibition and other results
            if st.button("Calculate"):
                results = pd.DataFrame({
                    "Konsentrasi (ppm)": concentrations,
                    "% Inhibisi": [(1 - a/blank)*100 if blank != 0 else 0 for a in absorbances]
                })
                st.table(results)

                # Linear regression
                coefficients = np.polyfit(concentrations, results["% Inhibisi"], 1)
                slope, intercept = coefficients
                regression_eq = f"y = {slope:.6f}x {'+' if intercept >= 0 else '-'} {abs(intercept):.6f}"
                st.markdown(f"**Persamaan Regresi:** {regression_eq}")

                # Graph
                fig, ax = plt.subplots()
                ax.plot(concentrations, results["% Inhibisi"], 'o-', label="Data Points")
                ax.plot(concentrations, slope * np.array(concentrations) + intercept, '-', label="Regresi")
                ax.set_xlabel("Konsentrasi (ppm)")
                ax.set_ylabel("% Inhibisi")
                ax.legend()
                st.pyplot(fig)

                # Display IC50 value
                ic50 = (50 -intercept) / slope if slope != 0 else None
                if ic50:
                    st.markdown(f"**IC50 Value:** {ic50:.6f} ppm")
                else:
                    st.markdown("**IC50 Value:** Tidak dapat dihitung.")

    elif method == "FRAP":
        st.markdown("<h1 style='color: orange; text-align: center;'>FRAP</h1>", unsafe_allow_html=True)
        blank = st.number_input("Input blanko", format="%.6f")
        num_samples = st.number_input("Berapa banyak sampel ingin diuji", min_value=0, step=1)

        if num_samples > 0:
            st.write("Masukkan konsentrasi (ppm) dan absorbansi untuk setiap sampel")
            cols = st.columns(2)
            concentrations = []
            absorbances = []
            for i in range(num_samples):
                with cols[0]:
                    concentrations.append(st.number_input(f"Input konsentrasi (ppm) sampel {i+1}", format="%.6f"))
                with cols[1]:
                    absorbances.append(st.number_input(f"Input absorbansi sampel {i+1}", format="%.6f"))

            # Calculate % reducing power and other results
            if st.button("Calculate"):
                results = pd.DataFrame({
                    "Konsentrasi (ppm)": concentrations,
                    "% Reducing Power": [(1 - (blank / a)) * 100 if blank != 0 else 0 for a in absorbances]
                })
                st.table(results)

                # Linear regression
                coefficients = np.polyfit(concentrations, results["% Reducing Power"], 1)
                slope, intercept = coefficients
                regression_eq = f"y = {slope:.6f}x {'+' if intercept >= 0 else '-'} {abs(intercept):.6f}"
                st.markdown(f"**Persamaan Regresi:** {regression_eq}")

                # Graph
                fig, ax = plt.subplots()
                ax.plot(concentrations, results["% Reducing Power"], 'o-', label="Data Points")
                ax.plot(concentrations, slope * np.array(concentrations) + intercept, '-', label="Regresi")
                ax.set_xlabel("Konsentrasi (ppm)")
                ax.set_ylabel("% Reducing Power")
                ax.legend()
                st.pyplot(fig)

                # Display EC50 value
                ec50 = (50 - intercept) / slope if slope != 0 else None
                if ec50:
                    st.markdown(f"**EC50 Value:** {ec50:.6f} ppm")
                else:
                    st.markdown("**EC50 Value:** Tidak dapat dihitung.")

    else:
        st.markdown("""<div style="text-align: center;">
                        <p>Kalkulator sedang dalam pengembangan.</p>
                        <span style="font-size: 50px;">😓</span>
                      </div>""", unsafe_allow_html=True)

# Main App Logic

if st.session_state.current_page == "Home":
    home()
elif st.session_state.current_page == "About Us":
    st.title("About Us")
    st.write("This is the About Us page.")
elif st.session_state.current_page == "Contact":
    st.title("Contact")
    st.write("This is the Contact page.")
elif "ATOX.CALC-" in st.session_state.current_page:
    method = st.session_state.current_page.split("-")[1]
    atox_calc_page(method)
else:
    page_2()


