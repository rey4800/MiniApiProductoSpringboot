# Login + CRUD de Usuarios | Spring Boot & Angular

Aplicación web que implementa un sistema de inicio de sesión básico y un CRUD (Crear, Leer, Actualizar y Eliminar) de usuarios.

El proyecto utiliza Spring Boot para desarrollar la API REST, Angular para la interfaz gráfica y Oracle o MySQL como sistema de gestión de bases de datos.

## Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| Java 17 o 21 | Lenguaje backend |
| Spring Boot | Desarrollo de API REST |
| Spring Data JPA | Persistencia de datos |
| Oracle / MySQL | Base de datos |
| Maven | Gestión de dependencias |
| Angular | Interfaz frontend |
| TypeScript | Lógica frontend |
| Postman | Pruebas de API |

---

# 1. Backend - Spring Boot

## 1.1. Crear el proyecto

Ingresar a [Spring Initializr](https://start.spring.io/) y configurar:

- Project: Maven
- Language: Java
- Java: 17 o 21
- Artifact: `login`

Dependencias:

- Spring Web
- Spring Data JPA

Para la conexión a la base de datos existen dos opciones:

- **Oracle:** agregar el controlador `ojdbc11` al archivo `pom.xml`.
- **MySQL:** seleccionar MySQL Driver desde Spring Initializr.

## 1.2. Configurar la conexión a la base de datos

Abrir:

`src/main/resources/application.properties`

### Opción A: Oracle

```properties
spring.datasource.url=jdbc:oracle:thin:@//localhost:1521/FREEPDB1
spring.datasource.username=prueba
spring.datasource.password=prueba123

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

### Opción B: MySQL

Primero crear la base de datos:

```sql
CREATE DATABASE login;
```

Después configurar:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/login
spring.datasource.username=root
spring.datasource.password=TU_PASSWORD

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

**Comprobación:** ejecutar la aplicación y verificar en la consola que aparezca:

```text
HikariPool-1 - Start completed
```

Esto indica que el pool de conexiones se inició correctamente.

## 1.3. Crear la entidad Usuario

Crear el archivo `Usuario.java`.

Esta clase representa la tabla `usuario` en la base de datos.

```java
import jakarta.persistence.*;

@Entity
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    private String correo;

    @Column(name = "clave")
    private String password;

    private String rol;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public String getCorreo() {
        return correo;
    }

    public void setCorreo(String correo) {
        this.correo = correo;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getRol() {
        return rol;
    }

    public void setRol(String rol) {
        this.rol = rol;
    }
}
```

**Explicación:**

- `@Entity`: indica que la clase representa una tabla.
- `@Id`: identifica la clave primaria.
- `@GeneratedValue`: permite generar automáticamente el identificador.
- `@Column`: permite especificar el nombre de una columna.
- Los atributos representan los datos que almacenará cada usuario.

## 1.4. Crear el repositorio

Crear `UsuarioRepository.java`.

Este repositorio permite realizar operaciones sobre la base de datos mediante Spring Data JPA.

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UsuarioRepository
        extends JpaRepository<Usuario, Long> {

    Usuario findByCorreoAndPassword(
        String correo,
        String password
    );
}
```

**Explicación:**

`JpaRepository<Usuario, Long>` proporciona métodos como:

- `findAll()`: obtener todos los usuarios.
- `findById()`: buscar por identificador.
- `save()`: guardar o actualizar.
- `deleteById()`: eliminar.

El método adicional:

```java
findByCorreoAndPassword()
```

Permite buscar un usuario que coincida con el correo y la contraseña.

Spring Data JPA genera la consulta a partir del nombre del método, sin necesidad de escribir SQL manualmente.

## 1.5. Crear el controlador

Crear `UsuarioController.java`.

Este controlador recibe las peticiones HTTP provenientes del frontend o de herramientas como Postman.

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/usuarios")
@CrossOrigin(origins = "http://localhost:4200")
public class UsuarioController {

    private final UsuarioRepository repo;

    public UsuarioController(UsuarioRepository repo) {
        this.repo = repo;
    }

    // LOGIN
    @PostMapping("/login")
    public ResponseEntity<Usuario> login(
            @RequestBody Usuario datos) {

        Usuario u = repo.findByCorreoAndPassword(
            datos.getCorreo(),
            datos.getPassword()
        );

        if (u == null) {
            return ResponseEntity.status(401).build();
        }

        return ResponseEntity.ok(u);
    }

    // LISTAR USUARIOS
    @GetMapping
    public List<Usuario> listar() {
        return repo.findAll();
    }

    // CREAR USUARIO
    @PostMapping
    public Usuario crear(@RequestBody Usuario u) {
        return repo.save(u);
    }

    // ACTUALIZAR USUARIO
    @PutMapping("/{id}")
    public Usuario actualizar(
            @PathVariable Long id,
            @RequestBody Usuario u) {

        u.setId(id);
        return repo.save(u);
    }

    // ELIMINAR USUARIO
    @DeleteMapping("/{id}")
    public void eliminar(@PathVariable Long id) {
        repo.deleteById(id);
    }
}
```

### Explicación de las anotaciones

| Anotación | Función |
|---|---|
| `@RestController` | Define un controlador REST. |
| `@RequestMapping` | Define la ruta principal. |
| `@CrossOrigin` | Permite solicitudes desde otro origen. |
| `@GetMapping` | Atiende peticiones GET. |
| `@PostMapping` | Atiende peticiones POST. |
| `@PutMapping` | Atiende peticiones PUT. |
| `@DeleteMapping` | Atiende peticiones DELETE. |
| `@RequestBody` | Recibe información del cuerpo de la petición. |
| `@PathVariable` | Obtiene valores de la URL. |

---

# 2. Probar el Backend con Postman

Antes de conectar Angular, comprobar que todos los endpoints funcionan.

La URL principal es:

```text
http://localhost:8080/api/usuarios
```

## 2.1. Crear el primer usuario

Método: `POST`

URL:

```text
http://localhost:8080/api/usuarios
```

En Postman seleccionar:

Body → raw → JSON.

```json
{
    "nombre": "Admin",
    "correo": "admin@correo.com",
    "password": "1234",
    "rol": "ADMIN"
}
```

La API guardará el usuario en la base de datos.

## 2.2. Iniciar sesión correctamente

Método: `POST`

URL:

```text
http://localhost:8080/api/usuarios/login
```

Body:

```json
{
    "correo": "admin@correo.com",
    "password": "1234"
}
```

Resultado esperado:

- Código HTTP: `200 OK`.
- Respuesta: información del usuario.

## 2.3. Probar un inicio de sesión incorrecto

Enviar el mismo correo con una contraseña diferente.

Resultado esperado:

```text
401 Unauthorized
```

## 2.4. Probar el CRUD

| Método | Endpoint | Función |
|---|---|---|
| GET | `/api/usuarios` | Listar usuarios |
| POST | `/api/usuarios` | Crear usuario |
| PUT | `/api/usuarios/{id}` | Actualizar usuario |
| DELETE | `/api/usuarios/{id}` | Eliminar usuario |

---

# 3. Frontend - Angular

## 3.1. Configurar HttpClient

En `app.config.ts` debe estar habilitado `provideHttpClient()`.

Ejemplo:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient()
  ]
};
```

## 3.2. Crear la lógica del componente

Archivo: `app.ts`

```typescript
import {
  Component,
  OnInit,
  signal,
  inject
} from '@angular/core';

import { FormsModule } from '@angular/forms';
import { HttpClient } from '@angular/common/http';

interface Usuario {
  id?: number;
  nombre: string;
  correo: string;
  password: string;
  rol: string;
}

@Component({
  imports: [FormsModule],
  selector: 'app-root',
  styleUrl: './app.css',
  templateUrl: './app.html',
})
export class App implements OnInit {

  http = inject(HttpClient);

  url = 'http://localhost:8080/api/usuarios';

  // LOGIN
  logueado = signal(false);
  mensaje = signal('');

  credenciales = {
    correo: '',
    password: ''
  };

  // CRUD
  usuarios = signal<Usuario[]>([]);

  usuario: Usuario = {
    nombre: '',
    correo: '',
    password: '',
    rol: ''
  };

  ngOnInit() {}

  // INICIAR SESIÓN
  login() {
    this.http.post<Usuario>(
      `${this.url}/login`,
      this.credenciales
    ).subscribe({

      next: () => {
        this.logueado.set(true);
        this.mensaje.set('');
        this.listar();
      },

      error: () => {
        this.mensaje.set(
          'Correo o contraseña incorrectos'
        );
      }

    });
  }

  // CERRAR SESIÓN
  salir() {
    this.logueado.set(false);

    this.credenciales = {
      correo: '',
      password: ''
    };
  }

  // LISTAR
  listar() {
    this.http.get<Usuario[]>(this.url)
      .subscribe(data => {
        this.usuarios.set(data);
      });
  }

  // CREAR O ACTUALIZAR
  guardar() {

    if (this.usuario.id) {

      this.http.put(
        `${this.url}/${this.usuario.id}`,
        this.usuario
      ).subscribe(() => {
        this.limpiar();
        this.listar();
      });

    } else {

      this.http.post(
        this.url,
        this.usuario
      ).subscribe(() => {
        this.limpiar();
        this.listar();
      });

    }
  }

  // EDITAR
  editar(u: Usuario) {
    this.usuario = { ...u };
  }

  // ELIMINAR
  eliminar(id: number) {
    this.http.delete(
      `${this.url}/${id}`
    ).subscribe(() => {
      this.listar();
    });
  }

  // LIMPIAR FORMULARIO
  limpiar() {
    this.usuario = {
      nombre: '',
      correo: '',
      password: '',
      rol: ''
    };
  }
}
```

### Explicación de los elementos principales

- `interface Usuario`: define la estructura de los datos de un usuario.
- `inject(HttpClient)`: permite utilizar el cliente HTTP para comunicarse con el backend.
- `signal(false)`: almacena el estado del inicio de sesión.
- `signal<Usuario[]>([])`: almacena la lista de usuarios.
- `ngOnInit()`: ciclo de vida del componente.
- `subscribe()`: permite recibir la respuesta de una petición HTTP.
- `next`: se ejecuta cuando la petición tiene éxito.
- `error`: se ejecuta cuando ocurre un error.

## 3.3. Crear la interfaz HTML

Archivo: `app.html`

```html
@if (!logueado()) {

  <!-- PANTALLA DE LOGIN -->

  <h1>Iniciar sesión</h1>

  <input
    [(ngModel)]="credenciales.correo"
    placeholder="Correo"
  />

  <input
    [(ngModel)]="credenciales.password"
    type="password"
    placeholder="Contraseña"
  />

  <button (click)="login()">Entrar</button>

  <p style="color: red">{{ mensaje() }}</p>

} @else {

  <!-- PANTALLA DEL CRUD -->

  <h1>Usuarios</h1>

  <button (click)="salir()">Cerrar sesión</button>

  <br /><br />

  <input
    [(ngModel)]="usuario.nombre"
    placeholder="Nombre"
  />

  <input
    [(ngModel)]="usuario.correo"
    placeholder="Correo"
  />

  <input
    [(ngModel)]="usuario.password"
    type="password"
    placeholder="Contraseña"
  />

  <input
    [(ngModel)]="usuario.rol"
    placeholder="Rol (ADMIN o USER)"
  />

  <button (click)="guardar()">Guardar</button>

  <button (click)="limpiar()">Cancelar</button>

  <table border="1">

    <tr>
      <th>ID</th>
      <th>Nombre</th>
      <th>Correo</th>
      <th>Rol</th>
      <th>Acciones</th>
    </tr>

    @for (u of usuarios(); track u.id) {

      <tr>
        <td>{{ u.id }}</td>
        <td>{{ u.nombre }}</td>
        <td>{{ u.correo }}</td>
        <td>{{ u.rol }}</td>

        <td>
          <button (click)="editar(u)">Editar</button>

          <button (click)="eliminar(u.id!)">
            Eliminar
          </button>
        </td>
      </tr>

    }

  </table>

}
```

### Explicación de Angular

- `@if`: permite mostrar el login o el CRUD dependiendo del estado de la sesión.
- `@for`: recorre la lista de usuarios y genera las filas de la tabla.
- `[(ngModel)]`: permite enlazar los campos del formulario con las propiedades del componente.
- `(click)`: ejecuta una función cuando se presiona un botón.
- `{{ }}`: muestra información del componente en el HTML.

---

# 4. Ejecutar el proyecto completo

Se necesitan dos terminales: una para Spring Boot y otra para Angular.

## 4.1. Iniciar el Backend

Abrir el proyecto Spring Boot y ejecutar:

```bash
mvn spring-boot:run
```

También se puede ejecutar directamente desde la clase principal de Spring Boot.

El servidor estará disponible en:

```text
http://localhost:8080
```

## 4.2. Iniciar el Frontend

Abrir una terminal dentro de la carpeta del proyecto Angular.

Instalar dependencias si todavía no están instaladas:

```bash
npm install
```

Iniciar el servidor:

```bash
ng serve
```

Abrir el navegador en:

```text
http://localhost:4200
```

## 4.3. Probar la aplicación

1. Iniciar el backend y verificar que esté conectado a la base de datos.
2. Crear el primer usuario desde Postman.
3. Iniciar Angular.
4. Abrir `http://localhost:4200`.
5. Introducir una contraseña incorrecta y comprobar que aparezca el mensaje de error.
6. Iniciar sesión con:

   - Correo: `admin@correo.com`
   - Contraseña: `1234`

7. Comprobar que aparece el CRUD de usuarios.
8. Crear, editar y eliminar usuarios.
9. Cerrar sesión e intentar iniciar sesión nuevamente.

---

# 5. Flujo de funcionamiento

El funcionamiento general de la aplicación es el siguiente:

```text
       ANGULAR
          |
          | Correo y contraseña
          v
    SPRING BOOT API
          |
          v
  UsuarioController
          |
          v
  UsuarioRepository
          |
          v
      BASE DE DATOS
          |
          v
   Validar credenciales
          |
     +----+----+
     |         |
   Existe   No existe
     |         |
     v         v
  HTTP 200   HTTP 401
     |         |
     v         v
  Mostrar    Mostrar
    CRUD      Error
```

---

# 6. Conceptos importantes para comprender el proyecto

| Concepto | Descripción |
|---|---|
| API REST | Permite la comunicación entre frontend y backend. |
| CRUD | Operaciones para crear, consultar, actualizar y eliminar registros. |
| JPA | API de persistencia para trabajar con bases de datos. |
| Repository | Interfaz encargada de acceder a los datos. |
| Controller | Recibe y procesa las solicitudes HTTP. |
| ResponseEntity | Permite controlar el código de respuesta HTTP. |
| Signal | Permite manejar estados reactivos en Angular. |
| HttpClient | Permite realizar solicitudes HTTP desde Angular. |
| CORS | Controla las solicitudes entre diferentes orígenes. |

# 7. Consideraciones de seguridad

**IMPORTANTE:** Este proyecto implementa un login básico con fines educativos y de práctica.

Actualmente:

- Las contraseñas se almacenan en texto plano.
- El CRUD no está protegido.
- El estado de sesión se controla únicamente desde Angular.
- No se implementa JWT.
- No se utiliza Spring Security.

Por lo tanto, cualquier persona que conozca los endpoints puede realizar solicitudes directamente a la API, incluso sin iniciar sesión.

En un sistema real se recomienda implementar:

- Spring Security.
- Cifrado de contraseñas mediante BCrypt.
- Autenticación mediante JWT.
- Protección de los endpoints.
- Autorización según los roles de usuario.

# 8. Autor

**Desarrollador:** [Reynaldo]

Proyecto desarrollado con fines educativos para practicar desarrollo backend con Java y Spring Boot, conexión a bases de datos Oracle/MySQL y consumo de API REST desde Angular.
