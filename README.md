# Script-para-Converter-Word-para-PDF
Converter arquivo Word para PDF

#!/bin/bash

# Script para converter arquivos Word (.docx) para PDF
# Requer: libreoffice ou unoconv instalado

# Função para verificar se um comando existe
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Verificar se o libreoffice ou unoconv está instalado
if command_exists libreoffice; then
    CONVERTER="libreoffice"
elif command_exists unoconv; then
    CONVERTER="unoconv"
else
    echo "Erro: É necessário ter LibreOffice ou unoconv instalado."
    echo "No Ubuntu/Debian: sudo apt install libreoffice"
    echo "Ou: sudo apt install unoconv"
    exit 1
fi

# Solicitar caminho dos arquivos Word
read -p "Digite o caminho completo onde estão os arquivos Word: " word_path

# Verificar se o caminho existe
if [ ! -d "$word_path" ]; then
    echo "Erro: O caminho '$word_path' não existe ou não é um diretório."
    exit 1
fi

# Solicitar nome da pasta de destino
read -p "Digite o nome da pasta para os arquivos PDF: " folder_name

# Criar pasta de destino
pdf_path="${word_path}/${folder_name}"
mkdir -p "$pdf_path"

# Verificar se a pasta foi criada
if [ ! -d "$pdf_path" ]; then
    echo "Erro: Não foi possível criar a pasta '$pdf_path'."
    exit 1
fi

echo "Convertendo arquivos Word para PDF..."
echo ""

# Contadores
total_files=0
converted_files=0
skipped_files=0

# Processar cada arquivo Word
for file in "$word_path"/*.docx; do
    if [ -f "$file" ]; then
        total_files=$((total_files + 1))
        filename=$(basename "$file" .docx)
        output_file="${pdf_path}/${filename}.pdf"
        
        # Verificar se o arquivo PDF já existe
        if [ -f "$output_file" ]; then
            echo "Pulando '${filename}.docx' - PDF já existe."
            skipped_files=$((skipped_files + 1))
            continue
        fi
        
        echo "Convertendo: ${filename}.docx"
        
        # Converter usando o método disponível
        if [ "$CONVERTER" = "libreoffice" ]; then
            libreoffice --headless --convert-to pdf --outdir "$pdf_path" "$file" > /dev/null 2>&1
        else
            unoconv -f pdf -o "$pdf_path" "$file" > /dev/null 2>&1
        fi
        
        # Verificar se a conversão foi bem-sucedida
        if [ -f "$output_file" ]; then
            converted_files=$((converted_files + 1))
            
            # Comparar tamanhos
            original_size=$(du -h "$file" | cut -f1)
            pdf_size=$(du -h "$output_file" | cut -f1)
            
            echo "  ✓ Sucesso! Original: ${original_size}, PDF: ${pdf_size}"
        else
            echo "  ✗ Falha na conversão de ${filename}.docx"
        fi
    fi
done

# Relatório final
echo ""
echo "=== RELATÓRIO ==="
echo "Arquivos processados: ${total_files}"
echo "Arquivos convertidos: ${converted_files}"
echo "Arquivos pulados: ${skipped_files}"
echo "Pasta de destino: ${pdf_path}"


