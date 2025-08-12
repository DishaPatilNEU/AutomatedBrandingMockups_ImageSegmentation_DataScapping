# Logo Placement Web Application

## Goal  
This web application allows users to input a website URL, automatically scrapes the company’s logo from that site, and applies the logo to various product images (shirt, cup, book, bottle, keychain) in multiple placement variants. The app generates images and PDFs displaying these logo placements for easy review and further use.

## Why  
- Automate the manual and time-consuming process of downloading logos and correctly placing them on product images.  
- Leverage AI-powered image segmentation models to accurately detect product regions for precise logo placement.  
- Generate multiple logo placement variants to offer diverse design options.  
- Provide a user-friendly web interface enabling instant logo placement visualization and PDF report downloads from just a website URL input.

## How  

### Overall Flow  
1. User inputs a URL in the web interface.  
2. Backend scrapes the URL to find logos, with fallback to Clearbit Logo API if needed.  
3. For each product type (shirt, cup, book, bottle, keychain):  
   - A pretrained segmentation model predicts the product’s region mask.  
   - Bounding boxes and centroids are computed from the mask.  
   - The logo is resized and placed at strategic variants within the detected region.  
   - Variants are saved as images and compiled into a multi-page PDF.  
4. The frontend displays all images and provides download links for PDFs.  
5. Static files are served via Flask routes, and ngrok enables easy public access for testing.

### Detailed Data Scraping (Logo Extraction) Steps  
1. **Receive Input URL:** User submits a website URL via the form.  
2. **Sanitize and Validate:** Clean input, verify it’s a valid HTTP/HTTPS URL.  
3. **Scrape Website for Logos:**  
   - Fetch HTML content using `requests`.  
   - Parse HTML with BeautifulSoup.  
   - Extract `<link>`, `<meta>`, and `<img>` tags potentially containing logos, e.g.:  
     - `<link rel="icon">` or `<link rel="shortcut icon">`  
     - `<meta property="og:image">`  
     - `<img>` tags with "logo" or "icon" in `alt` or `src` attributes.  
   - Normalize relative URLs to absolute URLs.  
   - Download candidate images, filtering by size and aspect ratio to exclude tiny or irrelevant images.  
   - Select best candidate logo by heuristics (e.g., largest square icon).  
4. **Fallback to Clearbit Logo API:**  
   - If no suitable logo found, request logo via `https://logo.clearbit.com/<domain>` API.  
   - Save the image locally for later use.  
5. **Save Logo Image:** Store the final logo in a dedicated folder with a consistent filename for use by other modules.

### Logo Application Process Summary  
- Load product test image and preprocess for model input.  
- Use pretrained PyTorch segmentation model to predict product mask.  
- Extract bounding box and centroid from largest mask contour using OpenCV and NumPy.  
- Load and optionally sharpen the scraped logo image.  
- Calculate resized logo dimensions based on bounding box and logo aspect ratio.  
- Generate multiple placement variants with different scales and offsets.  
- Paste logos onto copies of the base product image.  
- Save each variant and compile all into a PDF for easy review and sharing.

## Why This Approach Works Well  
- AI segmentation models provide precise product region detection, outperforming naive bounding boxes or manual annotation.  
- Multiple logo variants offer creative flexibility for design decisions.  
- Automated logo scraping removes manual steps, speeding up the workflow.  
- Clearbit API fallback ensures robustness when scraping is unsuccessful.  
- PDF reports allow straightforward review and distribution of logo placements.  
- Flask with ngrok enables quick deployment and testing, including in temporary environments like Google Colab.

