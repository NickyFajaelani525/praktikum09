
# UNSECURE DOCKERFILE
# BAD: Menggunakan base image besar dengan banyak package tidak diperlukan
FROM ubuntu:22.04

# BAD: Menjalankan apt-get tanpa membersihkan cache
RUN apt-get update
RUN apt-get install -y python3 python3-pip curl wget

# BAD: Meng-copy seluruh direktori
COPY . /app

# BAD: Tidak set WORKDIR
RUN cd /app && pip3 install -r requirements.txt

# BAD: Berjalan sebagai root
# Tidak ada USER directive

# BAD: Menyimpan credential di ENV
ENV DB_PASSWORD=supersecret123
ENV API_KEY=abc123xyz

EXPOSE 8080
CMD ["python3", "/app/app.py"]
