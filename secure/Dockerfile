# SECURE DOCKERFILE
# ✅ STAGE 1: Build stage - instal dependency
FROM python:3.12-alpine AS builder

#     GOOD: Set workdir dari awal
WORKDIR /build

#     GOOD: Copy hanya file yang diperlukan, bukan seluruh direktori
COPY requirements.txt .

# ✅ GOOD: Install dependency dan bersihkan cache dalam satu RUN
RUN pip install --no-cache-dir --user -r requirements.txt

# ✅ STAGE 2: Runtime stage - image final yang minimal
FROM python:3.12-alpine AS runtime

# ✅ GOOD: Buat non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

# ✅ GOOD: Set workdir
WORKDIR /app

# ✅ GOOD: Copy dependency dari build stage
COPY --from=builder /root/.local /home/appuser/.local

# ✅ GOOD: Copy hanya source code yang diperlukan
COPY --chown=appuser:appgroup app.py .

# ✅ GOOD: Credential dari environment (injected saat runtime, bukan hardcode)
# ENV DB_PASSWORD - JANGAN PERNAH hardcode di sini!

# ✅ GOOD: Jalankan sebagai non-root user
USER appuser

#     GOOD: Expose port yang dipakai
EXPOSE 8080

#     GOOD: Gunakan ENTRYPOINT + CMD untuk fleksibilitas
ENTRYPOINT ["python3"]
CMD ["app.py"]
