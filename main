import os
import base64
import requests
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from openai import OpenAI

# Initializes and returns the Selenium WebDriver.
def get_driver():
    options = webdriver.FirefoxOptions()
    driver = webdriver.Firefox(options=options)
    return driver

# Takes a full page screenshot of the given URL using the provided WebDriver.
def take_screenshot(driver, url):
    try:
        driver.get(url)
        WebDriverWait(driver, 10).until(lambda d: d.execute_script("return document.readyState") == "complete")
        driver.save_full_page_screenshot('./image.png')
    except Exception as e:
        print(f"Error taking screenshot: {e}")
    finally:
        driver.quit()

# Uploads an image to Imgur and returns the direct URL.
def upload_to_imgur(image_path, client_id):
    headers = {"Authorization": f"Client-ID {client_id}"}
    try:
        with open(image_path, "rb") as image_file:
            image_b64 = base64.b64encode(image_file.read()).decode('utf-8')
        data = {"image": image_b64, "type": "base64"}
        response = requests.post("https://api.imgur.com/3/upload", headers=headers, data=data)
        response.raise_for_status()
        return response.json()["data"]["link"]
    except Exception as e:
        raise Exception(f"Failed to upload image to Imgur: {e}")

# Sends the image URL to OpenAI for analysis and returns the response.
def get_openai_response(image_url):
    client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))
    try:
        response = client.chat.completions.create(
            model="gpt-4-vision-preview",
            messages=[
                {"role": "user", "content": "this is a screenshot of a landing page. tell me what to improve, focus on design only."},
                {"role": "system", "content": f"Analyzing image from URL: {image_url}"},
                {"role": "user", "content": {"image_url": image_url}}
            ],
            max_tokens=300,
        )
        return response.choices[0].message['content']
    except Exception as e:
        raise Exception(f"Error getting response from OpenAI: {e}")

# Main function to execute the script.
def main():
    imgur_client_id = os.getenv('IMGUR_CLIENT_ID')
    url = input("Enter a URL to evaluate: ")

    driver = get_driver()
    take_screenshot(driver, url)
    image_url = upload_to_imgur('./image.png', imgur_client_id)

    openai_response = get_openai_response(image_url)
    print(openai_response)

if __name__ == "__main__":
    main()
