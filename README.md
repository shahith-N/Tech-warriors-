# Tech-warriors-
import re
from collections import Counter
import requests
from io import BytesIO
from PIL import Image
import pytesseract
import spacy
import numpy as np

# --- Dummy Data and Models (For Demonstration) ---
# In a real-world scenario, these would be sophisticated APIs or large models.
FACT_CHECK_DATABASE = {
    "covid-19 vaccine causes infertility": "❌ False. Major health organizations state there is no evidence to support this claim.",
    "new currency note with satellite chip": "❌ False. The claim about embedded chips in new notes is a hoax.",
    "solar eclipse on this date will cause earthquakes": "❌ False. There is no scientific basis for this claim.",
}

# Load a simple Spacy model for NLP
try:
    nlp = spacy.load("en_core_web_sm")
except OSError:
    print("Downloading Spacy model 'en_core_web_sm'...")
    from spacy.cli import download
    download("en_core_web_sm")
    nlp = spacy.load("en_core_web_sm")

def get_fact_check_status(text):
    """Simulates a fact-checking database lookup."""
    text_lower = text.lower()
    for claim, status in FACT_CHECK_DATABASE.items():
        if claim in text_lower:
            return status
    return "✅ Not found in our database. Requires manual verification."

def analyze_text_manipulation(text):
    """Analyzes text for common manipulation techniques."""
    analysis = []

    # Check for emotional language
    emotional_keywords = re.findall(r'(urgently|shocking|must share|be vigilant|breaking news|viral)', text.lower())
    if emotional_keywords:
        analysis.append(f"⚠️ **Emotional Language:** Uses words like '{', '.join(set(emotional_keywords))}' to provoke an emotional response and encourage sharing.")

    # Check for urgency or call to action
    if re.search(r'share this|forward to everyone|don\'t delete|spread this message', text.lower()):
        analysis.append("⚠️ **Call to Action:** Encourages urgent sharing, a common tactic for spreading hoaxes quickly.")

    # Check for logical fallacies (simplified)
    if re.search(r'they want to hide this|the government is covering this up', text.lower()):
        analysis.append("⚠️ **Conspiracy/Appeal to Secrecy:** Suggests a hidden truth, preying on distrust of authority.")

    return analysis if analysis else ["✅ No obvious signs of manipulative language detected."]

def analyze_image_metadata(image_url):
    """Simulates checking image metadata and EXIF data."""
    try:
        response = requests.get(image_url)
        img = Image.open(BytesIO(response.content))
        exif_data = img._getexif()

        if exif_data:
            return "⚠️ **Image Metadata Found:** Image contains EXIF data (e.g., date, time, camera model), which could indicate authenticity. However, this data can be faked."
        else:
            return "✅ **No EXIF Metadata:** The image lacks EXIF data, which is common but can also be a sign of manipulation (e.g., screenshots often have no metadata)."
    except Exception as e:
        return f"❌ **Error analyzing image:** Could not process image metadata. {e}"

def analyze_text_in_image(image_url):
    """Simulates OCR to detect text in an image."""
    try:
        response = requests.get(image_url)
        img = Image.open(BytesIO(response.content))
        text = pytesseract.image_to_string(img)
        if text.strip():
            return f"⚠️ **Text Detected:** The image contains text that reads: '{text[:100]}...' (first 100 characters). This text can be further analyzed."
        return "✅ **No significant text detected**."
    except Exception as e:
        return f"❌ **Error performing OCR:** {e}"

def run_analysis():
    """Main function to run the misinformation analysis tool."""
    print("--- 🔬 Misinformation Analyzer Prototype ---")
    user_input = input("Enter text or an image URL to analyze: ").strip()

    if user_input.startswith(('http://', 'https://')) and any(ext in user_input for ext in ['.jpg', '.jpeg', '.png', '.gif']):
        print("\n--- 🖼️ Image Analysis ---")
        print(analyze_image_metadata(user_input))
        print(analyze_text_in_image(user_input))
        print("\n--- 🔍 Text in Image Analysis ---")
        # In a real tool, the OCR'd text would then be passed to the text analysis functions.
    else:
        print("\n--- 📄 Text Analysis ---")
        # Fact-checking
        fact_status = get_fact_check_status(user_input)
        print(f"**Fact Check Status:** {fact_status}")

        # Manipulation analysis
        manipulation_analysis = analyze_text_manipulation(user_input)
        print("\n**Manipulation Techniques:**")
        for item in manipulation_analysis:
            print(f"- {item}")

if __name__ == "__main__":
    run_analysis()
