[app.py](https://github.com/user-attachments/files/24898765/app.py)
import streamlit as st
import google.generativeai as genai

# 1. Configuração de Estética Premium
st.set_page_config(page_title="Festa Pro | Concierge", page_icon="🥂", layout="wide")

st.markdown("""
  [Uploading requirements.txt…]()
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
    html, body, [class*="st-"] { font-family: 'Inter', sans-serif; }
    .main { background-color: #F8FAFC; }
    div[data-testid="stMetric"] {
        background: white; border: 1px solid #E2E8F0;
        padding: 1.5rem !important; border-radius: 16px !important;
        box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05) !important;
    }
    .stButton>button {
        background: linear-gradient(135deg, #1E3A8A 0%, #3B82F6 100%);
        color: white; border: none; padding: 0.75rem; border-radius: 12px;
        font-weight: 600; transition: all 0.3s ease;
    }
    </style>
    """, unsafe_allow_html=True)

# 2. IA Engine com Adaptação Dinâmica (Anti-Erro 404)
@st.cache_resource
def get_safe_model_name(api_key):
    genai.configure(api_key=api_key)
    try:
        modelos = [m.name for m in genai.list_models() if 'generateContent' in m.supported_generation_methods]
        for m in modelos:
            if "gemini-1.5-flash" in m: return m
        return modelos[0]
    except:
        return "models/gemini-1.5-flash"

CHAVE_API = ""GOOGLE_API_KEY""
modelo_oficial = get_safe_model_name(CHAVE_API)

# 3. Sidebar Objetiva
with st.sidebar:
    st.markdown("<h1 style='color: #1E3A8A;'>🥂 Festa Pro</h1>", unsafe_allow_html=True)
    st.divider()
    cidade = st.text_input("📍 Localização", placeholder="Cidade/Bairro")
    tipo = st.selectbox("🎉 Evento", ["Churrasco", "Festa Infantil", "Casamento", "Jantar", "Coquetel"])
    pax = st.number_input("👥 Pessoas", min_value=1, value=20)
    orcamento = st.number_input("💰 Verba (R$)", min_value=100, value=1500)
    aluguel = st.toggle("🪑 Alugar Móveis?", value=True)
    st.divider()
    gerar = st.button("GERAR PLANEJAMENTO ✨")

# 4. Painel de Resultados
if gerar:
    if not cidade:
        st.warning("⚠️ Digite a localização para continuar.")
    else:
        st.markdown(f"## {tipo} em {cidade}")
        
        k1, k2, k3 = st.columns(3)
        k1.metric("Convidados", f"{pax}")
        k2.metric("Orçamento", f"R$ {orcamento}")
        k3.metric("Móveis", "Sim" if aluguel else "Não")
        
        st.divider()
        col_lista, col_links = st.columns([1.6, 1])
        
        with col_lista:
            st.subheader("🛒 Lista de Compras & Estrutura")
            placeholder = st.empty()
            
            # Prompt com foco em Unidades de Medida e Itens Específicos
            prompt = f"""
            Como cerimonialista expert, planeje {tipo} para {pax} pessoas em {cidade} com R$ {orcamento}.
            Móveis inclusos: {aluguel}.

            REGRAS DE OURO:
            1. Quantidades EXATAS (kg, pacotes, unidades, litros, gramas). Proibido usar "suficiente".
            2. Se Aluguel de Móveis for True, calcule 1 mesa para cada 4 pessoas e cadeiras individuais.
            3. Tabela Markdown: Item | Quantidade/Medida | Preço Estimado.
            4. Sem textos extras. Apenas a tabela.
            """
            
            try:
                model = genai.GenerativeModel(modelo_oficial)
                full_text = ""
                response = model.generate_content(prompt, stream=True)
                for chunk in response:
                    full_text += chunk.text
                    placeholder.markdown(full_text + "▌")
                placeholder.markdown(full_text)
            except Exception as e:
                st.error(f"Erro na geração: {e}")

        with col_links:
            st.subheader("🏢 Fornecedores Locais")
            st.caption(f"Personalizados para {tipo}")
            
            # LÓGICA DE BOTÕES DINÂMICOS
            cidade_url = cidade.replace(' ', '+')
            
            with st.container(border=True):
                st.markdown("**🍕 Suprimentos Principais**")
                
                if tipo == "Churrasco":
                    st.link_button("🥩 Buscar Açougues", f"https://www.google.com/maps/search/acougue+em+{cidade_url}")
                elif tipo == "Festa Infantil":
                    st.link_button("🥐 Salgadinhos e Doces", f"https://www.google.com/maps/search/salgadinhos+festa+em+{cidade_url}")
                elif tipo == "Casamento" or tipo == "Jantar":
                    st.link_button("🍽️ Buffet e Catering", f"https://www.google.com/maps/search/buffet+evento+em+{cidade_url}")
                else:
                    st.link_button("🛒 Supermercados Atacadistas", f"https://www.google.com/maps/search/supermercado+em+{cidade_url}")
                
                st.link_button("🥤 Depósitos de Bebidas", f"https://www.google.com/maps/search/distribuidora+bebidas+em+{cidade_url}")
            
            st.write("")
            
            with st.container(border=True):
                st.markdown("**📦 Estrutura e Decoração**")
                if aluguel:
                    st.link_button("🪑 Aluguel de Mesas/Cadeiras", f"https://www.google.com/maps/search/aluguel+mesas+cadeiras+em+{cidade_url}")
                
                if tipo == "Festa Infantil":
                    st.link_button("🏰 Aluguel de Brinquedos", f"https://www.google.com/maps/search/aluguel+brinquedos+festa+em+{cidade_url}")
                else:
                    st.link_button("🎈 Artigos de Decoração", f"https://www.google.com/maps/search/artigos+festa+em+{cidade_url}")

            st.divider()
            st.download_button("📩 Salvar Plano", full_text, file_name="festa_pro.txt", use_container_width=True)
