FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

EXPOSE 3000
ENV PUBLIC_URL=/kittygram
ENV HOST=0.0.0.0
ENV PORT=3000

# Копируем файлы в volume при запуске и запускаем dev server
CMD sh -c "mkdir -p /vol/static/kittygram && cp -r build/* /vol/static/kittygram/ 2>/dev/null || true && npm start"

