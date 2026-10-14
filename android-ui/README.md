# Hy-MT2 pe Android cu Termux

Această interfață web în limba română trimite traducerile către un server llama.cpp care rulează local. Nu descarcă sau rulează modelul de una singură; ai nevoie de un model GGUF compatibil și de `llama-server`.

## Pregătire

1. Instalează Termux dintr-o sursă de încredere, apoi actualizează pachetele și instalează uneltele:

   ```sh
   pkg update
   pkg install git cmake clang make python
   ```

2. Compilează llama.cpp pentru mediul Termux:

   ```sh
   git clone https://github.com/ggml-org/llama.cpp.git
   cd llama.cpp
   cmake -B build -DCMAKE_BUILD_TYPE=Release
   cmake --build build --config Release -j2
   ```

   Compilarea și inferența locală pot necesita mult timp, memorie și spațiu de stocare. Suportul depinde de arhitectura și resursele telefonului.

3. Descarcă un model **GGUF** Hy-MT2 compatibil cu llama.cpp de pe [Hugging Face](https://huggingface.co/tencent/Hy-MT2-1.8B-GGUF). README-ul principal menționează că fișierele GGUF Hy-MT2 depind de kernelul STQ din [PR #22836 llama.cpp](https://github.com/ggml-org/llama.cpp/pull/22836). Verifică dacă versiunea llama.cpp folosită include acel suport; simpla compilare nu garantează compatibilitatea.

## Pornește serverul și interfața

În prima sesiune Termux, pornește serverul cu calea către fișierul GGUF:

```sh
cd ~/llama.cpp
./build/bin/llama-server \
  --model ~/models/Hy-MT2-1.8B-GGUF.gguf \
  --host 127.0.0.1 \
  --port 8080 \
  --ctx-size 8192 \
  --jinja \
  --cors-allow-origin http://127.0.0.1:8000
```

Înlocuiește calea modelului cu locația reală a fișierului. Dacă opțiunea CORS nu este recunoscută de versiunea compilată, folosește o versiune llama.cpp care o suportă; nu dezactiva protecțiile și nu expune serverul pe `0.0.0.0`.

Într-o a doua sesiune Termux, din directorul `android-ui` al clonei proiectului:

```sh
cd /cale/catre/hyagent/android-ui
python -m http.server 8000 --bind 127.0.0.1
```

Deschide `http://127.0.0.1:8000` în browserul Android. Interfața se conectează implicit la `http://127.0.0.1:8080`; dacă folosești altă adresă sau port, deschide „Setări conexiune locală”.

## Note

- Textul este trimis doar către serverul local configurat în interfață.
- Interfața folosește limbi suportate de Hy-MT2; româna nu este listată între limbile suportate.
- Dacă telefonul nu are suficientă memorie sau spațiu, modelul poate porni greu ori deloc. O cuantizare mai mică poate reduce resursele necesare, dar poate afecta calitatea.
