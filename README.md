# AI-chatbot-with-ShipBob-API
Python script for the AI chatbot that integrates with email communication and the ShipBob API, as explained earlier. Ensure you configure all necessary credentials and dependencies.
Complete Python Script

import os
import requests
import openai
from flask import Flask, request, jsonify
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Flask application initialization
app = Flask(__name__)

# Configuration
OPENAI_API_KEY = "your_openai_api_key"  # Replace with your OpenAI API key
SHIPBOB_API_KEY = "your_shipbob_api_key"  # Replace with your ShipBob API key
SHIPBOB_API_BASE_URL = "https://api.shipbob.com"
EMAIL_USER = "your_email@example.com"  # Replace with your email
EMAIL_PASSWORD = "your_email_password"  # Replace with your email password

# Set OpenAI API key
openai.api_key = OPENAI_API_KEY


def generate_response(message):
    """
    Generate an AI response using OpenAI GPT.
    """
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": "You are a helpful customer support assistant."},
                {"role": "user", "content": message}
            ]
        )
        return response['choices'][0]['message']['content']
    except Exception as e:
        print(f"Error generating response: {e}")
        return "I'm sorry, I couldn't process your request. Please try again later."


def get_shipping_info(order_id):
    """
    Fetch shipping information from the ShipBob API.
    """
    try:
        url = f"{SHIPBOB_API_BASE_URL}/orders/{order_id}"
        headers = {"Authorization": f"Bearer {SHIPBOB_API_KEY}"}
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        return response.json()
    except Exception as e:
        print(f"Error fetching shipping info: {e}")
        return {"error": "Unable to retrieve shipping information at this time."}


def send_email(to_email, subject, body):
    """
    Send an email using an SMTP server.
    """
    try:
        msg = MIMEMultipart()
        msg['From'] = EMAIL_USER
        msg['To'] = to_email
        msg['Subject'] = subject

        msg.attach(MIMEText(body, 'plain'))

        with smtplib.SMTP('smtp.gmail.com', 587) as server:
            server.starttls()
            server.login(EMAIL_USER, EMAIL_PASSWORD)
            server.send_message(msg)

        print("Email sent successfully.")
    except Exception as e:
        print(f"Error sending email: {e}")


@app.route('/handle_inquiry', methods=['POST'])
def handle_inquiry():
    """
    API endpoint to handle customer inquiries.
    """
    data = request.json
    customer_email = data.get("email")
    message = data.get("message")
    order_id = data.get("order_id", None)

    # Generate an AI response
    response_text = generate_response(message)

    # If order ID is provided, fetch shipping info
    if order_id:
        shipping_info = get_shipping_info(order_id)
        if "error" not in shipping_info:
            response_text += f"\n\nHere is your shipping update:\n{shipping_info}"
        else:
            response_text += f"\n\n{shipping_info['error']}"

    # Send response via email
    if customer_email:
        send_email(customer_email, "Response to your inquiry", response_text)

    return jsonify({"response": response_text})


if __name__ == "__main__":
    app.run(debug=True)

Steps to Run the Script:

    Install Required Libraries:

pip install openai flask requests

Replace Placeholders:

    Replace your_openai_api_key with your OpenAI API key.
    Replace your_shipbob_api_key with your ShipBob API key.
    Replace your_email@example.com and your_email_password with your email credentials.

Run the Script:

python chatbot.py

Send POST Request to the Endpoint: Use a tool like Postman or curl to send a POST request to the /handle_inquiry endpoint:

    {
        "email": "customer@example.com",
        "message": "Can you update me on my order status?",
        "order_id": "12345"
    }

    Response:
        The system generates a response using OpenAI and optionally includes shipping details from ShipBob.
        Sends an email to the specified customer email.

Additional Notes:

    ShipBob API Documentation: Ensure you have access to ShipBob API documentation to customize endpoints and responses as required.

    Security:
        Avoid hardcoding sensitive credentials in production.
        Use environment variables or a secure secrets manager.

    Scalability:
        Deploy the Flask app to a cloud service (AWS, Google Cloud, or Heroku) for production use.
        Use a task queue (e.g., Celery) for handling email sending asynchronously.
