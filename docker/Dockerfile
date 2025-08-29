# Etapa 1: Base - Com as dependências já instaladas
FROM node:20-alpine3.19 AS base
WORKDIR /app

# Instalar dependências necessárias para o SWC no Alpine
RUN apk add --no-cache libc6-compat bash openssh-client docker-cli

# Copiar arquivos essenciais e instalar dependências
COPY package.json package-lock.json ./

# Define um argumento que pode ser passado durante o build
ARG NODE_ENV=development 
# Instala TODAS as dependências (incluindo devDependencies) se não for produção
RUN if [ "$NODE_ENV" = "production" ]; then npm install --omit=dev; else npm install; fi


# Etapa 2: Build (para produção e homologação)
# Esta etapa agora usa a base com as dependências já instaladas
FROM base AS build
WORKDIR /app
COPY . .
RUN npm run build 


# Etapa 3: Desenvolvimento - A etapa que nosso Dev Container vai usar
FROM base AS development
WORKDIR /app

# 👇 A MUDANÇA MAIS IMPORTANTE ESTÁ AQUI 👇
# Copia o resto do código-fonte, já definindo o 'node' como dono dos arquivos.
# Isso evita todos os nossos problemas de permissão.
COPY --chown=node:node . .

# Define o usuário padrão para este estágio.
# Isso significa que o CMD e qualquer comando futuro será executado como 'node'.
USER node

ENV NODE_ENV=development
EXPOSE 3000
CMD ["tail", "-f", "/dev/null"]


# Etapa 4: Produção (será usado com NGINX)
FROM base AS production
WORKDIR /app
USER node

# Copiar apenas o build e as dependências de produção
COPY --from=build /app/.next ./.next
# A node_modules já foi copiada na etapa 'base', não precisamos copiar de novo se a base for a mesma.
COPY --from=build /app/package.json ./package.json

CMD ["npm", "run", "start"]```