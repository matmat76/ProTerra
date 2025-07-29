cmake_minimum_required(VERSION 3.16)
project(ProTerra_Specification)

# Configuration des variables
set(MAIN_TEX_FILE "ebauches/specification.tex")
set(OUTPUT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/livrables")
set(SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}")

# Trouver LaTeX
find_package(LATEX REQUIRED)

if(NOT PDFLATEX_COMPILER)
    message(FATAL_ERROR "pdflatex not found!")
endif()

# Définir tous les fichiers sources LaTeX
file(GLOB_RECURSE TEX_SOURCES 
    "ebauches/*.tex"
    "ebauches/sections/*/*.tex"
)

# Définir les ressources (figures, schémas)
file(GLOB_RECURSE FIGURES 
    "figures/*"
    "schemas/*"
)

# Créer les répertoires de sortie s'ils n'existent pas
file(MAKE_DIRECTORY ${OUTPUT_DIR})
file(MAKE_DIRECTORY ${BUILD_DIR})

# Cible principale pour compiler la spécification
add_custom_target(specification ALL
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR}/figures ${BUILD_DIR}/figures
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR}/schemas ${BUILD_DIR}/schemas
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR}/ebauches ${BUILD_DIR}/ebauches
    COMMAND ${PDFLATEX_COMPILER} -interaction=nonstopmode -output-directory=${BUILD_DIR} ${MAIN_TEX_FILE}
    COMMAND ${PDFLATEX_COMPILER} -interaction=nonstopmode -output-directory=${BUILD_DIR} ${MAIN_TEX_FILE}
    COMMAND ${CMAKE_COMMAND} -E copy ${BUILD_DIR}/specification.pdf ${OUTPUT_DIR}/ProTerra_Specification.pdf
    WORKING_DIRECTORY ${SOURCE_DIR}
    DEPENDS ${TEX_SOURCES} ${FIGURES}
    COMMENT "Compilation de la spécification ProTerra vers livrables/"
    BYPRODUCTS ${OUTPUT_DIR}/ProTerra_Specification.pdf
)

# Cible pour générer uniquement le PDF final dans le dossier livrables (maintenant redondante)
add_custom_target(livrable
    DEPENDS specification
    COMMENT "PDF déjà disponible dans livrables/ via la cible specification"
)

# Cible pour nettoyer les fichiers auxiliaires LaTeX
add_custom_target(clean-latex
    COMMAND ${CMAKE_COMMAND} -E remove -f
        ${BUILD_DIR}/*.aux
        ${BUILD_DIR}/*.log
        ${BUILD_DIR}/*.out
        ${BUILD_DIR}/*.toc
        ${BUILD_DIR}/*.lot
        ${BUILD_DIR}/*.lof
        ${BUILD_DIR}/*.bbl
        ${BUILD_DIR}/*.blg
        ${BUILD_DIR}/*.synctex.gz
        ${BUILD_DIR}/*.fdb_latexmk
        ${BUILD_DIR}/*.fls
    WORKING_DIRECTORY ${SOURCE_DIR}
    COMMENT "Nettoyage des fichiers auxiliaires LaTeX"
)

# Cible pour nettoyer complètement (build seulement, préserve livrables)
add_custom_target(clean-all
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${BUILD_DIR}
    WORKING_DIRECTORY ${SOURCE_DIR}
    COMMENT "Nettoyage complet du répertoire de build (préserve livrables/)"
)

# Support pour BibTeX si un fichier .bib existe
file(GLOB BIB_FILES "*.bib" "ebauches/*.bib")
if(BIB_FILES AND BIBTEX_COMPILER)
    add_custom_target(specification-with-bib
        COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR}/figures ${BUILD_DIR}/figures
        COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR}/schemas ${BUILD_DIR}/schemas
        COMMAND ${CMAKE_COMMAND} -E copy_directory ${SOURCE_DIR}/ebauches ${BUILD_DIR}/ebauches
        COMMAND ${PDFLATEX_COMPILER} -interaction=nonstopmode -output-directory=${BUILD_DIR} ${MAIN_TEX_FILE}
        COMMAND ${BIBTEX_COMPILER} ${BUILD_DIR}/specification
        COMMAND ${PDFLATEX_COMPILER} -interaction=nonstopmode -output-directory=${BUILD_DIR} ${MAIN_TEX_FILE}
        COMMAND ${PDFLATEX_COMPILER} -interaction=nonstopmode -output-directory=${BUILD_DIR} ${MAIN_TEX_FILE}
        COMMAND ${CMAKE_COMMAND} -E copy ${BUILD_DIR}/specification.pdf ${OUTPUT_DIR}/ProTerra_Specification.pdf
        WORKING_DIRECTORY ${SOURCE_DIR}
        DEPENDS ${TEX_SOURCES} ${FIGURES} ${BIB_FILES}
        COMMENT "Compilation avec bibliographie"
    )
endif()

# Cible pour surveiller les changements (optionnel)
add_custom_target(watch
    COMMAND echo "Mode surveillance non implémenté. Utilisez 'make specification' pour recompiler."
    COMMENT "Pour surveiller les changements, utilisez un outil externe comme 'entr' ou 'fswatch'"
)

# Messages d'aide
add_custom_target(aide
    COMMAND echo "Cibles disponibles:"
    COMMAND echo "  specification     - Compile la specification"
    COMMAND echo "  livrable         - Compile et copie le PDF vers livrables/"
    COMMAND echo "  clean-latex      - Nettoie les fichiers auxiliaires LaTeX"
    COMMAND echo "  clean-all        - Nettoie completement le repertoire de build"
    COMMAND echo "  aide             - Affiche cette aide"
    COMMENT "Aide pour les cibles CMake"
)