# Cairo Kurulum (Linux) #

Öncelikle, Rust'ı kurmak için şu komutu kullanınız.
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Cairo'nun belirli bir sürümünü yüklemek isterseniz aşağıda verilen kodu tercih edebilirsiniz.
```
export CAIRO_GIT_TAG=v2.0.0
```
Bunu takiben, şu komut yardımıyla Cairo'yu sisteminize ekleyin.
```
curl -L https://github.com/franalgaba/cairo-installer/raw/main/bin/cairo-installer | bash
```
Eğer bir bağlantı problemi yaşıyorsanız, "curl" komutuna "-k" ekleyerek yeniden indirme işlemi gerçekleştirin.
```
curl -k -L https://github.com/franalgaba/cairo-installer/raw/main/bin/cairo-installer | bash
```
Derleme esnasında eğer bir hata mesajı alıyorsanız, şu komutları kullanarak cc bağlayıcısını kurun.
```
sudo apt-get install build-essential
```
Ardından, Cairo'yu kaldırın ve tekrar yüklemeye çalışın.
```
rm -fr ~/.cairo
curl -L https://github.com/franalgaba/cairo-installer/raw/main/bin/cairo-installer | bash
```
Hala bir problemle karşılaşıyorsanız, cc'nin ortam değişkenlerinin doğru konfigüre edildiğinden emin olmak için aşağıdaki adımları uygulayın.
```
echo $PATH
export PATH=$PATH:/path/to/cc
```
Cairo'yu tekrar yüklemek için ilgili komutları kullanın.
Caro için shell ortamınızı konfigüre edin.n 
Bash kullanıyorsanız:
```
echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.bashrc
echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.bashrc
echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.profile
echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.profile
echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.bash_profile
echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.bash_profile
```
Zsh kullanıyorsanız:
```
echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.zshrc
echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.zshrc
```
Ortam ayarlarınızı aktif hale getirmek için shell'i yeniden başlatın
```
exec "$SHELL"
```
Doğru bir kurulum yaptığınızdan emin olmak adına şu komutu çalıştırın ve "cairo-compile 2.0.0" çıktısını bekleyin.
```
cairo-compile --version
```

Scarb'ı sisteminize eklemek adına ilgili komutu kullanın.
```
curl --proto '=https' --tlsv1.2 -sSf https://docs.swmansion.com/scarb/install.sh | sh
```

Ardından shell ortamınızı bir kez daha yeniden başlatın.
```
exec "$SHELL"
```

Kurulumun başarılı olup olmadığını kontrol etmek için aşağıdaki komutu çalıştırın.
```scarb --version```
