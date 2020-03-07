# ePayco-Symfony
Componente para utilización de pasarela de pago integrada a ePayco desde Symfony


## Prerequisitos

1. Debe tener el siguiente software instalado en su estación de trabajo local:
   -  [PHP version >=5.5.9]
   -  [MySQL version 15.1, Distrib 10.1.38-MariaDB]
   -  [Symfony version >=3.4.0 hasta <= 3.4.17]
   -  [https://epayco.co/]
   
1. Actualizar tu controlador
   Antes de comenzar, debes actualizar el archivo DefaultController.php de tu proyecto `/src/YourBundle/Controller` con las acciones de confirmación, respuesta y nuevo pago online:

   ```conf
    /**
     * @Route("/payment/confirm", name="contabilidad_payment_confirm")
     * @Method({"POST"})
     */
    public function paymentConfirmAction()
    {
        /*En esta página se reciben las variables enviadas desde ePayco hacia el servidor.
        Antes de realizar cualquier movimiento en base de datos se deben comprobar algunos valores
        Es muy importante comprobar la firma enviada desde ePayco
        Ingresar  el valor de p_cust_id_cliente lo encuentras en la configuración de tu cuenta ePayco
        Ingresar  el valor de p_key lo encuentras en la configuración de tu cuenta ePayco
        */

        $p_cust_id_cliente = '67807';
        $p_key             = '491d6a0b6e992cf924edd8d3d088aff1';

        $x_ref_payco      = $_REQUEST['x_ref_payco'];
        $x_transaction_id = $_REQUEST['x_transaction_id'];
        $x_amount         = $_REQUEST['x_amount'];
        $x_currency_code  = $_REQUEST['x_currency_code'];
        $x_signature      = $_REQUEST['x_signature'];


        $signature = hash('sha256', $p_cust_id_cliente . '^' . $p_key . '^' . $x_ref_payco . '^' . $x_transaction_id . '^' . $x_amount . '^' . $x_currency_code);

        $x_response     = $_REQUEST['x_response'];
        $x_motivo       = $_REQUEST['x_response_reason_text'];
        $x_id_invoice   = $_REQUEST['x_id_invoice'];
        $x_autorizacion = $_REQUEST['x_approval_code'];

        //Validamos la firma
        if ($x_signature == $signature) {

            /*Si la firma esta bien podemos verificar los estado de la transacción*/

            $x_cod_response = $_REQUEST['x_cod_response'];

            switch ((int) $x_cod_response) {
                case 1:
                # code transacción aceptada
                echo 'transacción aceptada';
                break;

                case 2:
                # code transacción rechazada
                echo 'transacción rechazada';
                break;

                case 3:
                # code transacción pendiente
                echo 'transacción pendiente';
                break;

                case 4:
                # code transacción fallida
                echo 'transacción fallida';
                break;


                }
        } else {
            die('Firma no valida');
        }

        //return $this->render('@EGContabilidad/Default/payment.confirm.html.twig');
    }

    /**
     * @Route("/payment/response", name="contabilidad_payment_response")
     * @Method({"POST"})
     */
    public function paymentResponseAction()
    {
        return $this->render('@EGContabilidad/Default/payment.response.html.twig');
    }

    /**
     * @Route("/payment/new", name="contabilidad_payment_new")
     * @Method({"GET", "POST"})
     */
    public function paymentNewAction(Request $request)
    {
        return $this->render('@EGContabilidad/Default/payment.new.html.twig');
    }
   ```
