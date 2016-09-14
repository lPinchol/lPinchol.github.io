+++
categories = ["Dev", "lPinchol"]
date = "2016-04-11T17:06:32+02:00"
draft = false
featureimage = "img/avatar.png"
menu = ""
tags = ["lPinchol", "Dev"]
title = "C#: UTILIZAR MEGA API CLIENT"

+++

<center>![001](/img/Dev/mega_icono1.png)</center>


> Mega es el sucesor del servicio de archivos en la nube Megaupload, para aquellos a los que no les suena ninguno de los dos es como un dropbox, un servicio gratuito o de pago dependiendo del espacio que utilizamos para poder subir nuestros archivos. En mi opinión es un servicio increíble que nos permite almacenar copias de seguridad de archivos en la nube y/o compartirlos.

> En esta entrada mostraré como desde un proyecto en .Net en este caso con C# podéis subir archivos a mega. Además de subir archivos también podréis eliminar, modificar, mover y varias funciones más. Para comenzar tendréis que incorporar a vuestro proyecto varias referencias, si lo realizáis desde el administrador de paquetes nuget de Visual Studio ahorraréis tiempo y faena. Para abrir el administrador de paquetes nuget vamos a la barra principal de arriba y seleccionamos proyecto y administrar paquetes nuget. En la caja de búsqueda ponéis “mega.co.nz” sin comillas, y debe aparecer una librería que hará de cliente contra los servidores de mega, la seleccionáis y le dais a instalar. Además instalará automáticamente otra llamada Newtonsoft.Json que utilizará la librería mega para realizar algunas acciones.

> Tenemos el proyecto listo para empezar a utilizar los métodos y funciones que nos facilita mega en la clase que queramos, con ello vamos a la clase desde la que queramos que se suba el archivo y añadimos dos directivas using, la primera hace referencia a la api mega y la segunda nos permitirá crear un hilo y de este modo no bloquear nuestra aplicación mientras se realizan algunas tareas en línea:


``` C#
using CG.Web.MegaApiClient;
using System.Threading.Tasks;

```

> Ahora vamos con el método que se encargará de realizar la subida del archivo, actualizar la progressbar, y actualizar el texto de un label que irá informando al usuario de como va el proceso. El método es llamado dentro del evento “Shown” del form, y el form lo llamo mediante “ShowDialog()” desde el evento click de un botón de otro form, de esta manera es como una ventana emergente que no desaparecerá hasta que termine el proceso (entre 10 y 40 segundos). Para que quede mas claro, tenemos el “Form1” con un botón, este botón llamará al “Form2” mediante “ShowDialog()”, y dentro del evento “Shown” del “Form2” realizaremos la llamada al método encargado de subir el archivo “subirArchivoAMega()”.

``` C#
//Declaración de un hilo
 private Thread t;
 

 //Evento Shown que instancia y ejecuta el hilo (este evento se ejecuta después del Load).
 private void Form2_Shown(object sender, EventArgs e)
 {
     //Instancia un hilo para ejecutar el método "subirArchivoAMega".
     t = new Thread(subirArchivoAMega);
     t.Start();
 }
// Método que se encarga de subir el archivo a la nuve con la api "Mega".
private void subirArchivoAMega() {
    try {
          // Actualizar el label para informar al usuario.
          lblInfo.Invoke(new MethodInvoker(delegate { lblInfo.Text = "Conectando como 'tu_cuenta_de_email@gmail.com'"; }));
          // Aumentar la barra de progreso.
          progressBar1.Invoke(new MethodInvoker(delegate { progressBar1.PerformStep(); }));
          // Instancia de un cliente para conectar con mega.
          MegaApiClient cliente = new MegaApiClient();
          // Inicio de sesión con el cliente, pasando el correo y la contraseña de la cuenta mega a la que se sube el archivo.
          cliente.Login("tu_cuenta_de_email@gmail.com", "****************");

          // Aumentar la barra de progreso.
          progressBar1.Invoke(new MethodInvoker(delegate { progressBar1.PerformStep(); }));
          // Actualizar el label para informar al usuario.
          lblInfo.Invoke(new MethodInvoker(delegate { lblInfo.Text = "Obteniendo directorios..."; }));
          // Obtenemos los nodos (directorios/archivos) de la cuenta dentro de una variable.
          var nodos = cliente.GetNodes();

          // Actualizar el label para informar al usuario.
          lblInfo.Invoke(new MethodInvoker(delegate { lblInfo.Text = "Buscando carpeta 'Facturas'..."; }));
          // Comprobar si existe algún nodo (directorio) que se llame "Facturas" (en mi caso quiero subir el archivo a dicha carpeta).
          bool existe = cliente.GetNodes().Any(n => n.Name == "Facturas");

          // Crear dos nodos.
          Node root;
          Node carpeta;

         // Si el directorio facturas existe, se obtiene. Si no existe, se crea.
         if (existe == true) {
              // Actualizar el label para informar al usuario.
              lblInfo.Invoke(new MethodInvoker(delegate { lblInfo.Text = "Obteniendo la carpeta 'Facturas'...."; }));
              // Aumentar la barra de progreso.
              progressBar1.Invoke(new MethodInvoker(delegate { progressBar1.PerformStep(); }));

             // Obtenemos el directorio.
             carpeta = nodos.Single(n => n.Name == "Facturas");
         }else {
             // Actualizar label para informar al usuario.
             lblInfo.Invoke(new MethodInvoker(delegate { lblInfo.Text = "Creando carpeta 'Facturas'..."; }));
             // Aumentar la barra de progreso.
             progressBar1.Invoke(new MethodInvoker(delegate { progressBar1.PerformStep(); }));

             //Obtenemos el nodo raíz.
             root = nodos.Single(n => n.Type == NodeType.Root);
             // Creamos el directorio llamado "Facturas" en la raíz.
             carpeta = cliente.CreateFolder("Facturas", root);
         }

         // Aumentar la barra de progreso.
         progressBar1.Invoke(new MethodInvoker(delegate { progressBar1.PerformStep(); }));
         // Actualizar label para informar al usuario.
         lblInfo.Invoke(new MethodInvoker(delegate { lblInfo.Text = "Subiendo archivo..."; }));
         // Aumentar la barra de progreso.
         progressBar1.Invoke(new MethodInvoker(delegate { progressBar1.PerformStep(); }));

         // Subimos el archivo al directorio "Facturas", pasando la ruta del archivo a subir y el directorio de mega donde debe subirlo.
         Node archivo = cliente.Upload(@"C\:Mis facturas\factura1.pdf", carpeta);

         // Obtener el link de descarga del archivo subido por si se requiere para algo.
         Uri downloadUrl = cliente.GetDownloadLink(archivo);
         // Actualizar label para informar al usuario.
         lblInfo.Invoke(new MethodInvoker(delegate { lblInfo.Text = "Archivo subido con éxito."; }));

    }catch (Exception error){
        // Algo ha fallado, abortamos el subproceso.
        t.Abort();
        // Mensaje en pantalla para informar al usuario del error.
        MessageBox.Show("Error al intentar subir el archivo. " + error.Message, "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
    }

    // No se puede cerrar el form desde un subproceso ya que no es desde donde se ha creado. Con este código podemos cerrarlo.
    this.Invoke((MethodInvoker)delegate{
        this.Close();
    });
}

```

> Si os fijáis, hago constantemente referencias a elementos del form, el motivo es para que el usuario sepa en todo momento la evolución del proceso. Para acceder a los componentes de un form no podemos hacerlo directamente desde un subproceso, por eso utilizamos el método “Invoke” que ejecuta el delegado asignado al subproceso y nos permite comunicarnos con el usuario.

> Finalmente muestro mi diseño del form para que veáis el total de elementos a los que hago referencia en el método, aunque podéis implementar el diseño que os guste.

<center>![001](/img/Dev/megaapiclient_upload_file.png)</center>