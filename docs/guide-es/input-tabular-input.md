Recogida de tabular input
========================

A veces es necesario para manejar varios modelos del mismo tipo en una sola forma. Por ejemplo, los múltiples ajustes, donde
cada configuración se almacena como un par nombre-valor y está representado por un modelo `Setting` [registro activo] (db-active-record.md).
Este tipo de formulario también se refiere a menudo como "entrada tabular".
En contraste con esto, el manejo de diferentes modelos de diferente tipo, que se maneja en la sección
[Obtener datos para múltiples modelos] (input-multiple-models.md).

A continuación se muestra cómo implementar la entrada de tabla con Yii.

Hay tres situaciones diferentes para cubrir, que tienen que ser manejado ligeramente diferente:
- La actualización de un conjunto fijo de registros de la base de datos
- Creación de un conjunto dinámico de nuevos registros
- Actualización, Creación y borrado de los registros en una sola página

En contraste con los modelos de formularios individuales explicó antes, estamos trabajando con una variedad de modelos ahora.
Esta matriz se pasa a la vista para mostrar los campos de entrada para cada modelo en una tabla como estilo y nos
utilizará métodos de ayuda de [[yü \ Base \ Modelo]] que facilitan la carga y validación de varios modelos a la vez:

- [[yii\base\Model::loadMultiple()|Model::loadMultiple()]] datos de envío de carga en una gran variedad de modelos.
- [[yii\base\Model::validateMultiple()|Model::validateMultiple()]] valida una gran variedad de modelos.

### La actualización de un conjunto fijo de registros

Vamos a empezar con la acción del controlador:

```php
<?php

namespace app\controllers;

use Yii;
use yii\base\Model;
use yii\web\Controller;
use app\models\Setting;

class SettingsController extends Controller
{
    // ...

    public function actionUpdate()
    {
        $settings = Setting::find()->indexBy('id')->all();

        if (Model::loadMultiple($settings, Yii::$app->request->post()) && Model::validateMultiple($settings)) {
            foreach ($settings as $setting) {
                $setting->save(false);
            }
            return $this->redirect('index');
        }

        return $this->render('update', ['settings' => $settings]);
    }
}
```

En el código anterior estamos usando [[yii\db\ActiveQuery::indexBy()|indexBy()]] al recuperar modelos de la base de datos para rellenar una matriz indexada por los modelos de claves primarias.
Estos serán posteriormente utilizados para identificar los campos de formulario. [[yii\base\Model::loadMultiple()|Model::loadMultiple()]] llena múltiple
modelos con los datos del formulario que viene de la POST
y [[yii\base\Model::validateMultiple()|Model::validateMultiple()]] valida todos los modelos a la vez.
Como hemos validado nuestros modelos antes, usando `validateMultiple()`, ahora estamos pasando como `false`
un parámetro para [[yii\db\ActiveRecord::save()|save()]] para ejecutar la validación no dos veces.

Ahora el formulario que hay en vista `update`:

```php
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;

$form = ActiveForm::begin();

foreach ($settings as $index => $setting) {
    echo $form->field($setting, "[$index]value")->label($setting->name);
}

ActiveForm::end();
```

Aquí creamos una matriz inicial `$settings` que contiene un modelo por defecto para que siempre al menos un campo de texto será
visibles en la vista. Además añadimos más modelos para cada línea de entrada es posible que hayamos recibido.

En la vista que se puede usar javascript para añadir nuevas líneas de entrada de forma dinámica.

### La combinación de actualización, crear y eliminar en una sola página

> Nota: Esta sección está en desarrollo.
>
> Es aún no tiene contenido.

TBD
