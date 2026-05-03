# API Documentation - notas-api

## Principios REST aplicados

- Recursos definidos con sustantivos plurales: `notes`, `users`, `auth`.
- Verbos HTTP correctos: `GET`, `POST`, `PUT`, `DELETE`.
- URLs claras y consistentes:
  - `/api/v1/notes`
  - `/api/v1/notes/:id`
  - `/api/v1/notes/:id/shares`
  - `/api/v1/users`
  - `/api/v1/auth/login`
- Uso adecuado de códigos de estado HTTP:
  - `201 Created` para recursos creados.
  - `200 OK` para respuestas exitosas.
  - `400 Bad Request` para datos inválidos.
  - `401 Unauthorized` para credenciales faltantes o token inválido.
  - `403 Forbidden` cuando el usuario no tiene permisos.
  - `404 Not Found` cuando no existe el recurso.

## Autenticación

- Todas las rutas de `notes` requieren un header de autorización:

```http
Authorization: Bearer <token>
```

- El endpoint `POST /api/v1/auth/login` retorna un token JWT.
- El endpoint `POST /api/v1/users` solo puede ser usado por usuarios con rol `admin`.

## Endpoints

### Health Check

- `GET /api/health`
- Respuesta: `200 OK`

```json
{
  "status": "OK",
  "message": "API de notas activa"
}
```

---

### Autenticación

#### Iniciar sesión

- `POST /api/v1/auth/login`
- Body:
  - `email`: string
  - `password`: string
- Respuesta exitosa: `200 OK`

```json
{
  "token": "<jwt-token>"
}
```

---

### Usuarios

#### Registrar usuario (recurso REST)

- `POST /api/v1/users`
- Requiere token válido y rol `admin`.
- Body:
  - `name`: string
  - `email`: string
  - `password`: string
  - `role`: string (opcional, por defecto `user`)
- Respuesta exitosa: `201 Created`

```json
{
  "message": "User registered successfully"
}
```

---

### Notas

#### Listar notas del usuario autenticado

- `GET /api/v1/notes`
- Requiere token válido.
- Respuesta exitosa: `200 OK`

```json
[
  {
    "id": "...",
    "title": "Nota 1",
    "content": "Contenido",
    "imageUrl": "/uploads/...",
    "isPrivate": false,
    "password": null,
    "userId": "..."
  }
]
```

#### Crear nota

- `POST /api/v1/notes`
- Requiere token válido.
- Body:
  - `title`: string
  - `content`: string
  - `isPrivate`: boolean (opcional)
  - `password`: string (opcional)
  - `image` como multipart form-data (opcional)
- Respuesta exitosa: `201 Created`

#### Obtener nota por id

- `GET /api/v1/notes/:id`
- Requiere token válido.
- Respuesta exitosa: `200 OK`

#### Actualizar nota

- `PUT /api/v1/notes/:id`
- Requiere token válido.
- Solo el propietario puede actualizar.
- Body: campos actualizables como `title`, `content`, `isPrivate`, `password` y `image`.
- Respuesta exitosa: `200 OK`

#### Eliminar nota

- `DELETE /api/v1/notes/:id`
- Requiere token válido.
- Solo el propietario o un administrador puede borrar la nota.
- Respuesta exitosa: `200 OK`

#### Compartir nota

- `POST /api/v1/notes/:id/shares`
- Requiere token válido.
- Solo el propietario puede compartir la nota.
- Body:
  - `email`: string
- Respuesta exitosa: `200 OK`

```json
{
  "message": "Email sent successfully"
}
```

## Validaciones aplicadas

- `POST /api/v1/notes` valida que `title` y `content` existan.
- `POST /api/v1/notes/:id/shares` valida que exista `email` y que el recurso sea del usuario autenticado.
- `PUT` y `DELETE` en `notes/:id` validan propiedad y permisos.
- `POST /api/v1/auth/login` valida que `email` y `password` estén presentes.
