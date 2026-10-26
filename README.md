# proyecto_mvvm_room_camera

App Android (Kotlin + Jetpack Compose) con arquitectura **MVVM**, **Room**, **corrutinas**, **CameraX**, **Navigation** con menú lateral y **3 animaciones**.

## Abrir y ejecutar
1. Android Studio → *Open* → `C:\Users\SFP_Docente\AndroidStudioProjects\proyecto_mvvm_room_camera`
2. Esperar el *Gradle Sync* (Gradle 8.13, AGP 8.13.1, Kotlin 2.0.21, JDK 17).
3. Ejecutar en un dispositivo/emulador con cámara (minSdk 26).
4. Login de prueba: **admin@duoc.cl / admin123** (se crea al instalar, vía `RoomDatabase.Callback`).

## Estructura (MVVM)
```
app/src/main/java/com/example/app/
├── MainActivity.kt                (punto de entrada)
├── repository/                   (Model)
│   ├── data/      AppDatabase.kt, ProductDao.kt, UserDao.kt, TipoUsuarioDao.kt   (Room)
│   └── models/    Product.kt, User.kt, TipoUsuario.kt                           (Entidades)
├── viewmodels/    LoginViewModel.kt, ProductViewModel.kt, UserViewModel.kt, TipoUsuarioViewModel.kt, Validaciones.kt
├── navegations/   AppNavigation.kt (Routes, menú lateral, NavHost)
└── view/          theme/Theme.kt
    ├── utils/     CameraCapture.kt, PhotoBox.kt (componentes), Animations.kt (las 3 animaciones)
    └── ui/        LoginScreen.kt, HomeScreen.kt,
                   ProductListScreen.kt, ProductFormScreen.kt,
                   UserListScreen.kt, UserFormScreen.kt,
                   TipoUsuarioScreen.kt, CameraScreen.kt
```

## Requisitos cubiertos
| Requisito | Dónde |
|---|---|
| CRUD Productos (crear, listar, editar, eliminar) | `ProductDao`, `ProductViewModel`, `ProductListScreen`, `ProductFormScreen` |
| CRUD Usuarios + email único | `UserDao`, `UserViewModel`, `UserListScreen`, `UserFormScreen` |
| Tipos de usuario (FK en User) | `TipoUsuario` (`tipo_usuarios`: id PK, tipo_usuario ≤ 30), `User.tipoUsuarioId` FK RESTRICT; CRUD en `TipoUsuarioScreen`; selector en `UserFormScreen` |
| Login contra Room | `UserDao.login`, `LoginViewModel`, `LoginScreen` |
| Corrutinas | DAOs `suspend` + `Flow`; `viewModelScope.launch(Dispatchers.IO)`, `withContext`, `LaunchedEffect` |
| CameraX | `CameraCapture.kt` (Preview + ImageCapture + cambio de lente); usado en Cámara, foto de producto y foto de perfil |
| Menú y navegación | `ModalNavigationDrawer` + `TopAppBar` + `NavHost` con rutas y argumentos (`product_form?id=`) |
| Animación 1 | `AnimacionEntradaFormulario` (`view/utils/Animations.kt`): `AnimatedVisibility` fade + slide en formularios |
| Animación 2 | `Modifier.pulsoYRotacion()` (`view/utils/Animations.kt`): `rememberInfiniteTransition` en el logo del Login |
| Animación 3 | `navEnterTransition`… (`view/utils/Animations.kt`): `slideIntoContainer/slideOutOfContainer` en el `NavHost` |
| Permisos | `AndroidManifest.xml`: `CAMERA` + `uses-feature camera` (no obligatorio); permiso en tiempo de ejecución en `CameraCapture` |
| Dependencias | `gradle/libs.versions.toml` + `app/build.gradle.kts` |

## Notas
- Room usa **KSP** (reemplazo recomendado de kapt).
- Las fotos se guardan en el almacenamiento interno privado de la app (no visibles en Galería ni para otras apps, sin permisos de almacenamiento):
  `/data/data/com.example.app/files/fotos/{productos|usuarios|camara}/img_<timestamp>.jpg`.
  Se excluyen del respaldo (`res/xml/backup_rules.xml`, `res/xml/data_extraction_rules.xml`) y se borran al desinstalar.
  Para verlas en desarrollo: Android Studio > View > Tool Windows > Device Explorer.
- BD versión 2 con `fallbackToDestructiveMigration()`: al actualizar desde la v1 se recrea la BD. Datos iniciales: tipos Administrador/Agente/Supervisor y admin@duoc.cl (Administrador).
- Contraseñas en texto plano: solo con fines académicos.
