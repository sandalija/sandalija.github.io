---
layout: post
title: Accediendo al sistema de ficheros con React Native y Expo
date: 2022-04-05 22:22 +0200
---

Existen casos en los cuales queremos guardar ficheros o incluso imágenes en el sistema de archivos local para recuperarlos más tarde. Pongamos por ejemplo que queremos guardar una tarea con lo que vamos a hacer hoy. Para ello usamos el paquete `expo-file-system`.

Para identificar a un archivo, usaremos en todo momento su URI. Los tipos de archivos estan soportados están displonibles en [este enlace](https://docs.expo.dev/versions/latest/sdk/filesystem/#supported-uri-schemes-1). 
Las URIs deben ser de nuestra aplicación. Podemos obtener nuestro directorio  a traves de `expo-file-system`.

```typescript
import { documentDirectory } from "expo-file-system";
```

## Instala el paquete
Tan simple como 

```bash
yarn add expo-file-system
```

## Evaluar si existe el archivo
```getInfoAsync()``` nos devolverá un objeto con la propiedad `exists` indicando si un archivo existe o no para la URI pasada por parámetro
```typescript
export const fileSystemAccess = (uri: string): Promise<FileInfo> => {
  return getInfoAsync(uri);
};
```

## Crear un archivo
Podemos crear cualquier archivo a partir de una URI de nuestra app
```typescript
/**
 *
 * @param uri
 * @param content
 */
export const writeFile = async (
  uri: string,
  content: string
): Promise<void> => {
  return writeAsStringAsync(uri, content);
};

```

## Crear un archivo
Podemos obtener el contentido a través de su URI.

```typescript
/**
 *
 * @param uri
 */
export const readFile = async (uri: string): Promise<string> => {
  return readAsStringAsync(uri);
};
```

## Eliminar un archivo
De manera similar a leer un archivo, podemos acabar con él.
```typescript
/**
 *
 * @param uri
 */
export const removeFile = async (uri: string): Promise<void> => {
  return deleteAsync(uri);
};
```

## Vamos a unirlo todo
Finalmente, con todo lo que hemos visto,  podemos crear un componente que gestione el ciclo de vida de un fichero.


```typescript
// src/components/File.tsx
import React, { useEffect, useState } from "react";
import { StyleSheet, Text, TextInput, View } from "react-native";
import {
  fileSystemAccess,
  readFile,
  removeFile,
  writeFile,
} from "../lib/FileUtils";
import { Button } from "./Button"; // Custom button implementation

interface FileProps {
  fileName: string;
}

const File = (props: FileProps) => {
  const { fileName } = props;
  const [fileExists, setFileExists] = useState<boolean>(false);
  const [fileContent, setFileContent] = useState<string>("");
  const [input, setInput] = useState<string>("");

  const fetchFile = async () => {
    try {
      const result = await fileSystemAccess(fileName);
      setFileExists(result.exists);
      if (result.exists) {
        const content = await readFile(fileName);
        setFileContent(content);
      }
    } catch (e) {
      console.error(e);
    }
  };

  const handleWriteFile = async (input: string) => {
    try {
      await writeFile(fileName, input);
      setInput("");
      await fetchFile();
    } catch (e) {
      console.error(e);
    }
  };

  const handleRemoveFile = async () => {
    try {
      await removeFile(fileName);
      await fetchFile();
    } catch (e) {
      console.error(e);
    }
  };

  useEffect(() => {
    fetchFile();
  }, []);

  return (
    <View>
      {fileExists ? (
        <View>
          <Text>{fileContent}</Text>
          <TextInput
            onChangeText={(text) => setInput(text)}
            value={input}
            style={styles.input}
          />
          <Button
            title="Save"
            onPress={() => handleWriteFile(input)}
            color="#841584"
          />
          <Button title="Delete" onPress={handleRemoveFile} color="#841584" />
        </View>
      ) : (
        <View>
          <Text>What will you do today?</Text>
          <View style={styles.button}>
            <Button
              title="Create"
              onPress={() => handleWriteFile("Start creating!")}
              color="#841584"
            />
          </View>
        </View>
      )}
    </View>
  );
};

export default File;

const styles = StyleSheet.create({
  input: {
    height: 40,
    minWidth: 180,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
  button: {
    margin: 6,
  },
});

```


## Resultado

![Crear fichero](/assets/2022/04/05/create-image.jpeg)

![Edotar fichero](/assets/2022/04/05/edit-image.jpeg)
