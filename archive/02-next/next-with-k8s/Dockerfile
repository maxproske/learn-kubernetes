FROM node:20-alpine

ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT 80

WORKDIR /app
COPY . .

RUN npm install --omit=dev
RUN npm run build

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

USER nextjs

CMD ["npm", "run", "start"]
