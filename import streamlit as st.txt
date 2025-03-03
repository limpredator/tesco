import streamlit as st
import pandas as pd
import pytesseract
from PIL import Image
import io

def extract_text_from_image(image):
    """Extracts text from an uploaded image using OCR."""
    return pytesseract.image_to_string(image)

def process_text_to_dataframe(extracted_text):
    """Processes extracted text to extract grocery items, quantity, and price."""
    lines = extracted_text.split('\n')
    items = []
    
    for line in lines:
        parts = line.split()
        if len(parts) > 2:
            try:
                price = float(parts[-1].replace('£', ''))  # Extract price
                quantity = int(parts[-2])  # Extract quantity
                item_name = ' '.join(parts[:-2])  # Extract item name
                items.append([item_name, quantity, price])
            except ValueError:
                continue  # Skip lines that don't fit expected pattern
    
    return pd.DataFrame(items, columns=["Item Name", "Quantity", "Price (£)"])

# Streamlit App UI
st.title("Tesco Shopping List Extractor")
st.write("Upload an image of your Tesco basket, and it will generate an Excel file for you.")

uploaded_file = st.file_uploader("Upload Screenshot", type=["png", "jpg", "jpeg"])

if uploaded_file:
    image = Image.open(uploaded_file)
    st.image(image, caption="Uploaded Image", use_column_width=True)
    
    extracted_text = extract_text_from_image(image)
    df = process_text_to_dataframe(extracted_text)
    
    if not df.empty:
        st.write("### Extracted Shopping List")
        st.dataframe(df)
        
        output = io.BytesIO()
        with pd.ExcelWriter(output, engine='xlsxwriter') as writer:
            df.to_excel(writer, index=False)
        output.seek(0)
        
        st.download_button(label="Download Excel File", data=output, file_name="shopping_list.xlsx", mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")
    else:
        st.error("Could not extract shopping list. Please try a clearer image.")
