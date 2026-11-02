Tema 1 · El patrón Repository (o DAO)
1. ¿Qué es el patrón Repository y qué problema resuelve?
El patrón Repository actúa como una capa de mediación entre la lógica de dominio de la aplicación y la
infraestructura de datos. Abstrae el acceso a la base de datos simulando que las entidades persistidas forman parte
de una colección de objetos en memoria. Resuelve el problema del acoplamiento directo entre el código de negocio y
los motores de almacenamiento.
• Ejemplo RPG Manager: Permite que la entidad Guerrero persista su daño base e inventario mediante un contrato
genérico guardar(guerrero) o buscarPorId(id), sin importar si los datos se guardan en MariaDB o MongoDB.
Fuente oficial: Microsoft Learn — Repository Pattern Architecture Guidelines
2. Diferencia entre la capa de negocio (Service) y la capa de datos (Repository)
La capa Service coordina las operaciones del dominio e implementa las reglas de negocio del juego. La capa
Repository se limita a ejecutar operaciones CRUD de lectura y escritura en la base de datos.
• Ejemplo RPG Manager: Si un Mago intenta equipar un "Báculo Legendario", PersonajeService valida si el Mago
posee el nivel 20 requerido; si cumple, invoca a PersonajeRepository.actualizarInventario(idMago, item).
Fuente oficial: Martin Fowler — Separation of Concerns and Service Layer Pattern
3. ¿Por qué un Repository no debe validar reglas de negocio?
El repositorio debe respetar el Principio de Responsabilidad Única (SRP). Si validara reglas de negocio (como
comprobar los puntos de maná del Mago), se generaría duplicación de lógica, acoplamiento indeseado y dificultad
para realizar pruebas unitarias.
• Ejemplo RPG Manager: Si se cambia la regla para permitir que un Arquero lleve dos carcajes, la modificación se
realiza en PersonajeService sin tocar PersonajeRepository.
Fuente oficial: Baeldung — Single Responsibility Principle in Data Layers
Tema 2 · Contratos en Java y Python
1. Interfaces en Java y omisión de métodos
Una interface en Java define un contrato sintáctico formal. Si una clase concreta implementa la interfaz pero omite
algún método, el compilador genera un error de compilación impidiendo generar el bytecode.
• Ejemplo RPG Manager: Si la interfaz PersonajeRepository exige guardar(Personaje p) y la clase
MariaDBPersonajeRepository lo omite, el proyecto no compilará.
Fuente oficial: Oracle Java Documentation — Interfaces Specification
2. Clases abstractas con ABC y @abstractmethod en Python
En Python, las clases abstractas se crean heredando de abc.ABC y usando @abstractmethod. Si una subclase no
implementa un método abstracto e intenta instanciarse, Python lanza un TypeError en tiempo de ejecución.
• Ejemplo RPG Manager: Si PersonajeRepository(ABC) define @abstractmethod def obtener_por_id(self, id) y se
intenta invocar repo = PersonajeRepository(), Flask detendrá la ejecución.
Fuente oficial: Python Documentation — abc module specification
3. Semejanzas y diferencias entre contratos de Java y Python
Ambos mecanismos imponen un estándar polimórfico obligando a estructurar métodos específicos. La diferencia es
que Java valida el contrato en tiempo de compilación, mientras que Python lo efectúa en tiempo de ejecución
durante la instanciación.
• Ejemplo RPG Manager: En Java el error al omitir eliminar() salta en el IDE; en Python (Flask) salta al recibir la
petición HTTP que instancia la clase.
Fuente oficial: Real Python — Official Guide to PyABC and Polymorphism
Tema 3 · Inyección de dependencias por constructor
1. Programar contra una interfaz y no contra una implementación
Significa vincular los componentes de alto nivel a contratos abstractos en lugar de a clases concretas. Permite
intercambiar motores tecnológicos o infraestructuras sin alterar el código de la lógica de negocio.
• Ejemplo RPG Manager: PersonajeService depende de PersonajeRepository, lo que permite cambiar de MariaDB a
MongoDB sin modificar la lógica de los ataques del Guerrero.
Fuente oficial: Martin Fowler — Inversion of Control Containers and Dependency Injection
2. Inyección de dependencias por constructor
Es la técnica donde las dependencias requeridas por un componente se suministran directamente a través de su
método constructor al ser instanciado, en lugar de instanciarse internamente.
• Código de ejemplo (Java): public PersonajeService(PersonajeRepository repo) { this.repo = repo; }
Fuente oficial: Baeldung — Constructor Dependency Injection Pattern in Java
3. Ventajas de la inyección por construct
Otorga bajo acoplamiento y alta testeabilidad. Permite realizar pruebas unitarias inyectando repositorios simulados
(Mocks) en memoria sin necesidad de levantar bases de datos reales durante las pruebas automatizadas.
• Ejemplo RPG Manager: Durante las pruebas del daño de bola de fuego del Mago, se pasa un
MockPersonajeRepository que no se conecta a MariaDB.
Fuente oficial: Refactoring Guru — Dependency Injection Pattern
Tema 4 · Configuración y secretos
1. Lectura de archivos de configuración (.properties en Java y .ini en Python)
En Java se utiliza la clase java.util.Properties mediante ClassLoader. En Python se emplea la librería nativa
configparser con la clase ConfigParser().
• Ejemplo RPG Manager: Permite leer la URL de conexión del servidor (db.url=jdbc:mariadb://localhost:3306/rpg_db)
en Javalin o la clave del servidor en Flask.
Fuente oficial: Python Documentation — configparser module
2. Propósito de .gitignore y archivos previamente rastreados
El archivo .gitignore impide que archivos con credenciales o binarios se incluyan en el control de versiones. Si un
archivo fue subido en un commit anterior, incluirlo después en .gitignore no lo elimina; requiere ejecutar git rm
--cached para desrastrearlo.
• Ejemplo RPG Manager: Evita la exposición pública del archivo credenciales_mariadb.properties en GitHub.
Fuente oficial: Git Official Documentation — gitignore specification
3. Caso real de filtración de llaves y credenciales en GitHub
En 2019, análisis académicos identificaron más de 100,000 repositorios públicos en GitHub con claves expuestas de
AWS y credenciales de BD. Bots maliciosos escanean GitHub e inyectan scripts en segundos, generando ataques de
secuestro de datos (Ransomware) y facturas elevadas.
• Consecuencia en RPG Manager: Si se sube la clave de MariaDB, bots pueden borrar la tabla de inventario del
Guerrero o secuestrar el servidor.
Fuente oficial: Wired — The Leaky Bucket of Public GitHub Repositories Study
Tema 5 · MariaDB desde el código
1. Concepto de JDBC, rol del Driver y su equivalente en Python
JDBC es la API estándar de Java para ejecutar sentencias relacionales. El Driver convierte las llamadas genéricas al
protocolo nativo de MariaDB. En Python, las librerías equivalentes son PyMySQL o mariadb-connector-python.
• Ejemplo RPG Manager: El driver permite ejecutar SELECT * FROM personajes WHERE tipo='Arquero' en el puerto
3306.
Fuente oficial: MariaDB Connector Documentation
2. Inyección SQL y mitigación con PreparedStatement y parámetros %s
Ocurre al concatenar texto ingresado por el usuario dentro de sentencias SQL. Se evita usando PreparedStatement
(Java) o tuplas cursor.execute(sql, (param,)) con %s (Python), enviando los datos separados de la estructura lógica.
• Ejemplo con O'Brien: Si el nombre es O'Brien' OR '1'='1, la concatenación altera la consulta devolviendo todos los
registros; los parámetros parametrizados la tratan como un texto literal inofensivo.
Fuente oficial: OWASP Foundation — SQL Injection Prevention Cheat Sheet
3. Transacciones y función de commit() en PyMySQL
Una transacción agrupa sentencias bajo el principio de atomicidad (ACID). El método commit() aplica los cambios en
la BD. En PyMySQL, el autocommit está desactivado por defecto; si no se llama a commit(), las operaciones de
modificación se descartan.
• Consecuencia en RPG Manager: Al agregar un objeto al inventario del Mago, si no se llama conexion.commit(), el
item desaparece al cerrar el juego.
Fuente oficial: PyMySQL Documentation — Transactions and Commit
4. Cierre de conexiones, try-with-resources y with
Las conexiones no cerradas provocan fugas de recursos (connection leaks) agotando los sockets del sistema. En
Java se gestiona con try-with-resources y en Python mediante el bloque de contexto with.
• Ejemplo en Java: try (Connection conn = DriverManager.getConnection(url)) { ... } garantiza el cierre automático.
Fuente oficial: Oracle Java Tutorials — The try-with-resources Statement
Tema 6 · MongoDB desde el código
1. Estructura interna de un ObjectId en MongoDB
El ObjectId es una clave única de 12 bytes compuesta por: 4 bytes de marca de tiempo (timestamp), 5 bytes
aleatorios únicos por proceso/máquina, y 3 bytes de contador incremental.
• Ejemplo RPG Manager: Garantiza que si dos jugadores crean un Guerrero al mismo tiempo en servidores distintos,
no existan colisiones de claves.
Fuente oficial: MongoDB Manual — ObjectId Specification
2. Equivalencia de documentos BSON con estructuras en lenguaje
Un documento BSON almacena datos en parejas Clave-Valor. En Java equivale a un HashMap o Document, mientras
que en Python es idéntico a un diccionario nativo dict.
• Ejemplo en Python: guerrero_doc = {"nombre": "Conan", "clase": "Guerrero", "inventario": ["Espada"]}
Fuente oficial: MongoDB Docs — BSON Types Specification
3. Diferencias entre MongoDB Local, MongoDB Atlas y mongoplayground.net
MongoDB Local corre instalado en la máquina (localhost:27017). MongoDB Atlas es la versión administrada en la
nube (DBaaS). mongoplayground.net es un simulador web interactivo y no puede conectarse con programas
externos de Java o Python
• Uso en RPG Manager: El código Java/Python únicamente puede conectarse a MongoDB Local o Atlas mediante su
URI de conexión.
Fuente oficial: MongoDB Atlas Manual
Tema 7 · ORM (Object-Relational Mapping)
1. Concepto de ORM
Un ORM es una técnica y librería que mapea tablas relacionales a clases y registros a objetos orientados a objetos,
automatizando las operaciones SQL sin escribir consultas a mano.
• Ejemplo RPG Manager: Convierte filas de MariaDB directamente a la clase Mago m = new Mago().
Fuente oficial: Red Hat — What is Object-Relational Mapping?
2. Librerías de ORM en Java y Python
En Java el estándar principal es Hibernate (JPA / Jakarta Persistence). En Python se utiliza SQLAlchemy o Django
ORM.
Fuente oficial: Hibernate Docs / SQLAlchemy Docs
3. Ventajas y desventajas de un ORM
Ventaja: Alta velocidad de desarrollo y prevención nativa de inyección SQL. Desventaja: Penalización de rendimiento
en consultas complejas (problema de las N+1 consultas) y consumo elevado de memoria.
• Ejemplo en RPG Manager: Para traer 1,000 Personajes, un ORM mal configurado podría lanzar 1,001 consultas
individuales ralentizando el servidor.
Fuente oficial: Martin Fowler — OrmHate

Sección Especial · Preguntas de Verificación en Clase
Respuestas Rápidas para Evaluación al Azar
1. ¿Qué pasaría si el nombre fuera O'Brien y el SQL estuviera concatenado?
Se genera un SyntaxError al romper las comillas o se abre una vulnerabilidad grave de Inyección SQL. Se soluciona
usando sentencias parametrizadas con PreparedStatement (Java) o %s (Python).
2. ¿Por qué la clave no puede quedar en GitHub y qué hacer si ya se subió?
Bots escanean llaves públicas para ataques cibernéticos. Si ya se subió, incluirlo en .gitignore no basta; debe eliminarse
del historial con git rm --cached, hacer commit y cambiar la clave en MariaDB de inmediato.
3. ¿Quién decide si el Service usa MariaDB o MongoDB?
El desarrollador en la clase principal de configuración mediante Inyección de Dependencias por Constructor. Al
PersonajeService no le importa el motor, solo recibe una implementación de PersonajeRepository.
4. ¿Qué devuelve un dict de Python si se pide una clave inexistente?
Lanza una excepción de tipo KeyError. Para evitar fallos en el servidor Flask se debe emplear el método seguro
dict.get('clave', None).