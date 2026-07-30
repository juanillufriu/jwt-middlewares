# Laboratorio NestJS — Middlewares, JWT, Roles y SQLite

<p align="left">
<img src="https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white" /> <img src="https://img.shields.io/badge/JavaScript-323330?style=for-the-badge&logo=javascript&logoColor=F7DF1E" />
</p>

Repositorio del proyecto: ejercicios-rest

Repositorio GitHub del proyecto: https://github.com/juanillufriu/ejercicios-rest

---

# Descripción General

Este proyecto consiste en la implementación completa de un laboratorio backend utilizando NestJS.

Durante el desarrollo se implementaron:

- Middlewares personalizados
- Logger HTTP
- Timing Middleware
- ConfigModule
- Variables de entorno
- SQLite
- TypeORM
- bcrypt/bcryptjs
- JWT
- Passport
- JwtStrategy
- JwtAuthGuard
- RolesGuard
- Decorators personalizados
- Bootstrap admin
- Rutas protegidas
- Arquitectura modular NestJS

Además, se documentaron:

- errores encontrados,
- problemas de configuración,
- incompatibilidades en Windows,
- correcciones realizadas,
- y validaciones funcionales.

---

# Tecnologías Utilizadas

- NestJS
- TypeScript
- SQLite
- TypeORM
- Passport
- JWT
- bcryptjs
- Node.js
- Express

---

# Instalación del Proyecto

```bash
npm install
```

---

# Dependencias Instaladas

```bash
npm install @nestjs/config @nestjs/jwt @nestjs/passport @nestjs/typeorm passport passport-jwt typeorm sqlite3 better-sqlite3 bcryptjs
```

Dependencias de desarrollo:

```bash
npm install -D @types/passport-jwt
```

---

# Variables de Entorno

Archivo:

```txt
.env
```

Contenido:

```env
BCRYPT_COST=12
SQLITE_DATABASE=./database.sqlite
JWT_SECRET=super_secret_key_demo
JWT_EXPIRES_SEC=3600
```

---

# Ejercicio 1 — Configuración Inicial del Proyecto

## Modificación 1 — Instalación inicial de dependencias

### Archivos modificados

- package.json

### Descripción

Se instalaron las dependencias base necesarias para el funcionamiento del laboratorio.

### Código utilizado

```bash
npm install
```

### Justificación

Permite preparar el entorno NestJS y descargar las librerías necesarias.

### Pruebas funcionales

```bash
npm run start:dev
```

Resultado esperado:

```txt
Nest application successfully started
```

---

# Ejercicio 2 — Middlewares

## Modificación 2 — Creación de carpeta common

### Archivos modificados

```txt
src/common/
```

### Descripción

Se creó la carpeta para almacenar componentes reutilizables.

### Justificación

Mantener arquitectura modular y escalable.

---

## Modificación 3 — LoggerMiddleware

### Archivo modificado

```txt
src/common/middlewares/logger.middleware.ts
```

### Código

```ts
import {
  Injectable,
  NestMiddleware,
} from '@nestjs/common';

import {
  NextFunction,
  Request,
  Response,
} from 'express';

@Injectable()
export class LoggerMiddleware
  implements NestMiddleware
{
  use(
    req: Request,
    res: Response,
    next: NextFunction,
  ) {
    console.log(
      `[${new Date().toISOString()}] ${req.method} ${req.originalUrl}`,
    );

    next();
  }
}
```

### Justificación

Permite registrar las peticiones HTTP.

### Pruebas funcionales

Al realizar requests:

```txt
[2026-05-25T14:37:56.325Z] POST /auth/register
```

---

## Modificación 4 — TimingMiddleware

### Archivo modificado

```txt
src/common/middlewares/timing.middleware.ts
```

### Código

```ts
import {
  Injectable,
  NestMiddleware,
} from '@nestjs/common';

import {
  NextFunction,
  Request,
  Response,
} from 'express';

@Injectable()
export class TimingMiddleware
  implements NestMiddleware
{
  use(
    req: Request,
    res: Response,
    next: NextFunction,
  ) {
    const start = Date.now();

    res.on('finish', () => {
      const ms = Date.now() - start;

      res.setHeader(
        'X-Response-Time',
        `${ms}ms`,
      );
    });

    next();
  }
}
```

### Justificación

Permite medir tiempo de respuesta.

---

## Modificación 5 — Registro global de middlewares

### Archivo modificado

```txt
src/app.module.ts
```

### Código

```ts
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(
      LoggerMiddleware,
      TimingMiddleware,
    )
    .forRoutes('*');
}
```

### Justificación

Aplica middlewares globalmente.

---

# Ejercicio 3 — ConfigModule y Variables de Entorno

## Modificación 6 — ConfigModule global

### Archivo modificado

```txt
src/app.module.ts
```

### Código

```ts
ConfigModule.forRoot({
  isGlobal: true,
  envFilePath: ['.env'],
})
```

### Justificación

Permite usar variables de entorno globales.

---

## Modificación 7 — Archivo .env

### Archivo creado

```txt
.env
```

### Código

```env
BCRYPT_COST=12
SQLITE_DATABASE=./database.sqlite
JWT_SECRET=super_secret_key_demo
JWT_EXPIRES_SEC=3600
```

### Justificación

Separación entre configuración y código.

---

# Ejercicio 4 — SQLite y TypeORM

## Modificación 8 — Configuración TypeORM

### Archivo modificado

```txt
src/app.module.ts
```

### Código

```ts
TypeOrmModule.forRoot({
  type: 'sqlite',
  database:
    process.env.SQLITE_DATABASE ??
    './database.sqlite',
  synchronize: true,
  autoLoadEntities: true,
})
```

### Justificación

Configura SQLite y TypeORM.

---

## Modificación 9 — Creación UserEntity

### Archivo creado

```txt
src/users/user.entity.ts
```

### Código

```ts
import {
  Column,
  Entity,
  PrimaryGeneratedColumn,
} from 'typeorm';

import { UserRole } from './user-role.enum';

@Entity('users')
export class UserEntity {
  @PrimaryGeneratedColumn('uuid')
  id!: string;

  @Column({ unique: true })
  email!: string;

  @Column({ select: false })
  passwordHash!: string;

  @Column({
    type: 'text',
    default: UserRole.USER,
  })
  role!: UserRole;
}
```

### Justificación

Define entidad persistente de usuarios.

---

## Modificación 10 — Enum UserRole

### Archivo creado

```txt
src/users/user-role.enum.ts
```

### Código

```ts
export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
}
```

### Justificación

Define roles del sistema.

---

# Ejercicio 5 — DTOs

## Modificación 11 — RegisterDto

### Archivo creado

```txt
src/auth/dto/register.dto.ts
```

### Código

```ts
export class RegisterDto {
  email!: string;
  password!: string;
}
```

### Justificación

Modela request de registro.

---

## Modificación 12 — LoginDto

### Archivo creado

```txt
src/auth/dto/login.dto.ts
```

### Código

```ts
export class LoginDto {
  email!: string;
  password!: string;
}
```

### Justificación

Modela request de login.

---

# Ejercicio 6 — AuthService

## Modificación 13 — Implementación AuthService

### Archivo creado

```txt
src/auth/auth.service.ts
```

### Descripción

Se implementó:

- registro de usuarios,
- hashing de passwords,
- bootstrap admin,
- login,
- validación de credenciales,
- generación JWT.

### Justificación

Centraliza lógica de autenticación.

### Pruebas funcionales

Registro exitoso:

```json
{
  "id":"089a75fb-50d7-4c7b-9cca-a900c0ff8ef2",
  "email":"admin@test.com",
  "role":"admin"
}
```

---

# Ejercicio 7 — AuthController

## Modificación 14 — Endpoints Auth

### Archivo creado

```txt
src/auth/auth.controller.ts
```

### Código

```ts
@Post('register')
register(@Body() dto: RegisterDto) {
  return this.authService.register(dto);
}

@Post('login')
login(@Body() dto: LoginDto) {
  return this.authService.login(dto);
}
```

### Justificación

Expone endpoints HTTP.

---

# Ejercicio 8 — Passport y JWT

## Modificación 15 — Configuración JwtModule

### Archivo modificado

```txt
src/auth/auth.module.ts
```

### Código

```ts
JwtModule.registerAsync({
  imports: [ConfigModule],
  inject: [ConfigService],
  useFactory: (cfg: ConfigService) => ({
    secret: cfg.getOrThrow<string>(
      'JWT_SECRET',
    ),
    signOptions: {
      expiresIn: Number(
        cfg.get<string>(
          'JWT_EXPIRES_SEC',
        ) ?? '3600',
      ),
    },
  }),
})
```

### Justificación

Configura JWT dinámicamente.

---

## Modificación 16 — JwtStrategy

### Archivo creado

```txt
src/auth/strategies/jwt.strategy.ts
```

### Código

```ts
import { Injectable } from '@nestjs/common';

import { PassportStrategy } from '@nestjs/passport';

import {
  ExtractJwt,
  Strategy,
} from 'passport-jwt';

import { ConfigService } from '@nestjs/config';

@Injectable()
export class JwtStrategy extends PassportStrategy(
  Strategy,
) {
  constructor(cfg: ConfigService) {
    super({
      jwtFromRequest:
        ExtractJwt.fromAuthHeaderAsBearerToken(),

      ignoreExpiration: false,

      secretOrKey:
        cfg.getOrThrow<string>(
          'JWT_SECRET',
        ),
    });
  }

  validate(payload: any) {
    return {
      userId: payload.sub,
      role: payload.role,
    };
  }
}
```

### Justificación

Valida JWT automáticamente.

---

## Modificación 17 — JwtAuthGuard

### Archivo creado

```txt
src/auth/guards/jwt-auth.guard.ts
```

### Código

```ts
import { Injectable } from '@nestjs/common';

import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard(
  'jwt',
) {}
```

### Justificación

Protege rutas autenticadas.

---

# Ejercicio 9 — Roles

## Modificación 18 — Roles Decorator

### Archivo creado

```txt
src/common/decorators/roles.decorator.ts
```

### Código

```ts
import { SetMetadata } from '@nestjs/common';

import { UserRole } from '../../users/user-role.enum';

export const ROLES_KEY = 'roles';

export const Roles = (
  ...roles: UserRole[]
) => SetMetadata(ROLES_KEY, roles);
```

### Justificación

Define metadata de roles.

---

## Modificación 19 — RolesGuard

### Archivo creado

```txt
src/common/guards/roles.guard.ts
```

### Justificación

Valida autorización basada en roles.

---

## Modificación 20 — Endpoint protegido

### Archivo modificado

```txt
src/users/controllers/users.controller.ts
```

### Código

```ts
@Get()
@UseGuards(
  JwtAuthGuard,
  RolesGuard,
)
@Roles(UserRole.ADMIN)
findAllUsers() {
  return {
    message:
      'Ruta protegida solo para administradores',
  };
}
```

### Justificación

Protege endpoint admin-only.

---

# Ejercicio 10 — AuthModule

## Modificación 21 — Implementación AuthModule

### Archivo modificado

```txt
src/auth/auth.module.ts
```

### Descripción

Se configuró:

- PassportModule
- JwtModule
- JwtStrategy
- Guards
- TypeORM repository

### Justificación

Centralizar autenticación.

---

## Modificación 22 — Importar AuthModule

### Archivo modificado

```txt
src/app.module.ts
```

### Código

```ts
imports: [
  AuthModule,
]
```

### Justificación

Habilita autenticación global.

---

## Modificación 23 — Configuración UsersModule

### Archivo modificado

```txt
src/users/users.module.ts
```

### Código

```ts
TypeOrmModule.forFeature([
  UserEntity,
])
```

### Justificación

Permite inyectar Repository<UserEntity>.

---

## Modificación 24 — Corrección imports users.controller.ts

### Problema encontrado

Los imports estaban incorrectos debido a la estructura real del proyecto.

### Corrección

```ts
import { JwtAuthGuard } from '../../auth/guards/jwt-auth.guard';
```

### Justificación

Resolver errores TS2307.

---

## Modificación 25 — Corrección imports users.module.ts

### Problema encontrado

Ruta incorrecta del controller.

### Corrección

```ts
import { UsersController } from './controllers/users.controller';
```

### Justificación

Resolver errores de módulos.

---

# Ejercicio 11 — Correcciones de Errores

## Modificación 26 — Error TS2304 POST/auth/register

### Problema

El archivo auth.controller.ts contenía texto descriptivo en lugar de TypeScript.

### Error

```txt
Cannot find name 'POST'
```

### Solución

Reemplazo completo del controller.

### Justificación

Restaurar código válido.

---

## Modificación 27 — Error jwt-auth.guard.ts faltante

### Problema

No existía:

```txt
src/auth/guards/jwt-auth.guard.ts
```

### Solución

Creación del guard.

### Justificación

Resolver errores TS2307.

---

## Modificación 28 — Error body vacío RegisterDto {}

### Problema

El body JSON llegaba vacío.

### Evidencia

```txt
RegisterDto {}
```

### Causa

Problemas de escaping usando curl en Windows/Git Bash.

### Solución

Uso correcto de curl.exe:

```bash
curl.exe -X POST "http://localhost:3000/auth/register" -H "Content-Type: application/json" -d "{\"email\":\"admin@test.com\",\"password\":\"123456\"}"
```

### Justificación

Permitir parseo correcto del JSON.

---

## Modificación 29 — Error trim undefined

### Problema

```txt
Cannot read properties of undefined (reading 'trim')
```

### Causa

dto.email llegaba undefined.

### Solución

Corrección del body JSON enviado.

### Justificación

Garantizar datos válidos.

---

## Modificación 30 — Error bcrypt en Windows

### Problema

bcrypt nativo presenta incompatibilidades frecuentes en Windows.

### Solución

```bash
npm uninstall bcrypt
npm install bcryptjs
```

### Código modificado

```ts
import * as bcrypt from 'bcryptjs';
```

### Justificación

Mayor compatibilidad multiplataforma.

---

# Ejercicio 12 — Validaciones Funcionales

## Modificación 31 — Registro exitoso

### Prueba funcional

```bash
curl.exe -X POST "http://localhost:3000/auth/register" -H "Content-Type: application/json" -d "{\"email\":\"admin@test.com\",\"password\":\"123456\"}"
```

### Resultado

```json
{
  "id":"089a75fb-50d7-4c7b-9cca-a900c0ff8ef2",
  "email":"admin@test.com",
  "role":"admin"
}
```

---

## Modificación 32 — Login exitoso

### Prueba funcional

```bash
curl.exe -X POST "http://localhost:3000/auth/login" -H "Content-Type: application/json" -d "{\"email\":\"admin@test.com\",\"password\":\"123456\"}"
```

### Resultado esperado

```json
{
  "access_token":"eyJhbGciOiJIUzI1NiIs..."
}
```

---

## Modificación 33 — Ruta protegida

### Prueba funcional

```bash
curl.exe http://localhost:3000/users -H "Authorization: Bearer TU_TOKEN"
```

### Resultado esperado

```json
{
  "message":"Ruta protegida solo para administradores"
}
```

---

# Ejercicio 13 — Verificaciones Finales

## Modificación 34 — Verificación build

### Comando

```bash
npm run build
```

### Resultado esperado

Compilación sin errores.

---

## Modificación 35 — Verificación start:dev

### Comando

```bash
npm run start:dev
```

### Resultado esperado

```txt
Nest application successfully started
```

---

## Modificación 36 — Actualización .gitignore

### Código agregado

```txt
.env
.env.*
!.env.example
database.sqlite
```

### Justificación

Evitar subir secretos y base SQLite.

---

## Modificación 37 — Resultado Final

### El proyecto implementa exitosamente

- Middlewares personalizados
- ConfigModule
- Variables de entorno
- SQLite
- TypeORM
- bcryptjs
- JWT
- Passport
- JwtStrategy
- JwtAuthGuard
- RolesGuard
- Decorators personalizados
- Bootstrap admin
- Endpoints auth
- Rutas protegidas
- Arquitectura modular NestJS

---

# Problemas Encontrados y Soluciones

| Problema | Solución |
|---|---|
| TS2304 POST/auth/register | Reemplazo completo auth.controller.ts |
| jwt-auth.guard.ts faltante | Creación del archivo correcto |
| RegisterDto {} | Corrección curl Windows |
| trim undefined | JSON body inválido |
| bcrypt Windows | Migración a bcryptjs |
| Imports incorrectos | Corrección rutas relativas |
| curl Linux en CMD | Uso correcto curl.exe |
| main.ts alterado | Restauración bootstrap Nest |

---

# Estructura Final del Proyecto

```txt
src/
 ├── auth/
 │    ├── dto/
 │    ├── guards/
 │    ├── strategies/
 │    ├── auth.controller.ts
 │    ├── auth.module.ts
 │    └── auth.service.ts
 │
 ├── common/
 │    ├── decorators/
 │    ├── guards/
 │    └── middlewares/
 │
 ├── users/
 │    ├── controllers/
 │    ├── user.entity.ts
 │    ├── user-role.enum.ts
 │    └── users.module.ts
 │
 ├── products/
 ├── categories/
 │
 └── app.module.ts
```

---

# Ejecución Final

## Levantar servidor

```bash
npm run start:dev
```

## Registro

```bash
curl.exe -X POST "http://localhost:3000/auth/register" -H "Content-Type: application/json" -d "{\"email\":\"admin@test.com\",\"password\":\"123456\"}"
```

## Login

```bash
curl.exe -X POST "http://localhost:3000/auth/login" -H "Content-Type: application/json" -d "{\"email\":\"admin@test.com\",\"password\":\"123456\"}"
```

## Ruta protegida

```bash
curl.exe http://localhost:3000/users -H "Authorization: Bearer TU_TOKEN"
```

---

# Conclusión

El laboratorio fue completado exitosamente implementando autenticación JWT, middlewares, TypeORM, SQLite y control de roles en NestJS.

Además, se documentaron todos los problemas encontrados durante el desarrollo, incluyendo errores de compilación, incompatibilidades de Windows y problemas de parseo JSON.
