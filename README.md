# T'inquiète 
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <fstream>
using namespace std;

/**
 * @brief affichVect
 * @param[in] tab : tableau (vecteur) a affchicher
 */
template <typename typeDelaCellule>
void affichVect (const vector<typeDelaCellule> & tab){
    for (size_t i (0); i < tab.size(); ++i)
        cout << tab[i] << '\t';
    cout << endl;
}


/**
 * @brief affichMatrice
 * @param[in] mat matrice a afficher
 */
template <typename typeDelaCellule>
void affichMatrice (const vector<vector<typeDelaCellule>> & mat){
    cout << string (mat.size(), '-') << endl;
    for (const vector<typeDelaCellule> & uneLigne : mat){
        for (const typeDelaCellule & uneCellule : uneLigne)
            cout << uneCellule << '\t';
        cout << endl;
    }
    cout << string (mat.size(), '-') << endl;
}

/**
 * @brief typeCelluleULAM : type de la cellule d'une matrice
 */
typedef size_t typeCelluleULAM;
/**
 * @brief matrice : une matrice, comme son nom l'indique
 */
typedef vector<vector<typeCelluleULAM>> matrice;

/**
 * @brief generNombrePremier : génère tous les nombres premiers dans l'intervale [0 .. nbPremier[
 * la méthode est décrite ici : //https://fr.wikipedia.org/wiki/Crible_d%27%C3%89ratosth%C3%A8ne
 * @param[in] nbPremier valeur maximale des nombres premiers à générer
 * @return tableau des nombres premiers plus petit (au sens strict) que nbPremier
 */
vector <typeCelluleULAM> generNombrePremier (const size_t & nbPremier){
    vector <typeCelluleULAM> vecNbPremiers (nbPremier);
    for (size_t i (1); i < vecNbPremiers.size (); ++i)
        vecNbPremiers[i] = i;

     //suppression des éléments non premiers selon la méthode du crible d'Ératosthène
    for (size_t i (2); i < vecNbPremiers.size (); ++i){
        if (0 == vecNbPremiers[i]) continue;
        size_t j (vecNbPremiers[i]*vecNbPremiers[i]);
        while (j < vecNbPremiers.size()){
            vecNbPremiers[j] = 0;
            j = j + vecNbPremiers[i];
        }
    }

    //tassement du tableau (ie on vire les cases ayant pour valeur 0)
    typeCelluleULAM posAInserer (2);
    for (size_t i (2); i < vecNbPremiers.size (); ++i){
        if (0 == vecNbPremiers[i]) continue;
        vecNbPremiers[posAInserer++] = vecNbPremiers[i];
    }
    //on supprime les élements innutiles
    vecNbPremiers.resize(posAInserer);
    return vecNbPremiers;
}


/**
 * @brief affichMatriceULAM : on affiche, dans un terminal, un espace
 * si la valeur de la cellule vaut 0
 * sinon (si c'est un nombre premier), on affcihe un '#'
 * @param[in] mat : mattrice à afficher selon la méthode d'ULAM
 */
template <typename typeDelaCellule>
void affichMatriceULAM (const vector<vector<typeDelaCellule>> & mat){
    for (const vector<typeDelaCellule> & uneLigne : mat){
        for (const typeDelaCellule & uneCellule : uneLigne)
            cout << (uneCellule == 0 ?  ' ' : '#');
        cout << endl;
    }
}

const string estPremier ("255 255 255 ");
const string nEstPasPremier ("0 0 0 ");
/**
 * @brief ULAMversPPM : enregistre / sauvergarde une matrice ("au format ULAM") dans un fichier
 * de type PPM (https://fr.wikipedia.org/wiki/Portable_pixmap) en utilisant des string.
 * Tous les nombres (premiers et non premiers) ont le même code
 * @param[in] mat : matrice à convertir en fichier PPM
 */
void ULAMversPPM(const matrice & mat){
    ofstream sortie ("ULAM_"  + to_string(mat.size()) + ".ppm");
    sortie << "P3" << endl;
    sortie << mat.size() << " " << mat.size() << endl;
    sortie << "255" << endl;
    for (const vector<typeCelluleULAM> & uneLigne : mat){
        for (const typeCelluleULAM & uneCellule : uneLigne)
            sortie << (uneCellule == 0 ?  nEstPasPremier : estPremier);
        sortie << endl;
    }
}

const short niveauRougeNonPremier (0);
const short niveauVertNonPremier (0);
const short niveauBleuNonPremier (0);
const short niveauRougePremier (255);
const short niveauVertPremier (255);
const short niveauBleuPremier (255);

/**
 * @brief ULAMversPPMV2 enregistre / sauvergarde une matrice ("au format ULAM") dans un fichier
 * de type PPM (https://fr.wikipedia.org/wiki/Portable_pixmap) en utilisant des entiers.
 *  On peut jouer sur la valeur injectée pour les nombres premiers et non premiers
 * @param[in] mat : matrice à convertir en fichier PPM
 */
void ULAMversPPMV2 (const matrice & mat){
    ofstream sortie ("ULAM_"  + to_string(mat.size()) + "_V2.ppm");
    sortie << "P3" << endl;
    sortie << mat.size() << " " << mat.size() << endl;
    sortie << "255" << endl;

    for (const vector<typeCelluleULAM> & uneLigne : mat){
        for (const typeCelluleULAM & uneCellule : uneLigne) {
            if (0 == uneCellule)
                sortie << niveauRougeNonPremier << ' '
                       << niveauVertNonPremier << ' '
                       << niveauBleuNonPremier << ' ';
            else
                sortie << niveauRougePremier << ' '
                       << niveauVertPremier << ' '
                       << niveauBleuPremier << ' ';
        }

        sortie << endl;
    }
}

/**
 * @brief transformeMatEscargotEnUlam :  tranforme une matrice en mettant des 0 si la valeur de la case
 * courante n'appartient pas à tableauDesNombresAConserver
 * @param [in,out] mat : matrice dans laquelle on souhaite "
 * @param [in] tabDeSuppression : tableau des nombres à conserver
 */
void transformeMatEscargotEnUlam (matrice & mat,
                                 const vector <typeCelluleULAM> tableauDesNombresAConserver){
    for (vector<typeCelluleULAM> & uneLigne : mat){
        for (typeCelluleULAM & uneCellule : uneLigne){
            if (!binary_search(tableauDesNombresAConserver.begin(),
                               tableauDesNombresAConserver.end(), uneCellule))
                uneCellule = 0;
        }
    }
}


/**
 * @brief genrerMatriceEscargot : genère la matrice en forme d'escrgot comme vu dans le TP #12
 * @param mat[in,out] : matrice générée
 * @note la matrice est supposée être dimensionnée et d'ordre impair avant l'appel à cette
 * procédure
 */
void genrerMatriceEscargot (matrice & mat){
    size_t posLigne (mat.size()/2), posColonne (mat.size()/2);
    size_t valAInser (0);
    mat [posLigne][posColonne] = 0;
    unsigned nbPosAInsererDansCeTour (2);
    ++posColonne;
    //le -1 (à la fin de la conduition du while)
    //est là parce qu'on commence à partir de 0 ;)
    while (valAInser < mat.size() * mat.size() - 1){
        //deplacement en haut
        for (unsigned i (0); i < nbPosAInsererDansCeTour; ++i)
            mat[posLigne--][posColonne] = ++valAInser;
        //on remet la position d'insertion au bon endroit
        ++posLigne;
        --posColonne;

        //deplacement à gauche
        for (unsigned i (0); i < nbPosAInsererDansCeTour; ++i)
            mat[posLigne][posColonne--] = ++valAInser;
        //on remet la position d'insertion au bon endroit
        ++posLigne;
        ++posColonne;

        //deplacement en bas
        for (unsigned i (0); i < nbPosAInsererDansCeTour; ++i)
            mat[posLigne++][posColonne] = ++valAInser;
        //on remet la position d'insertion au bon endroit
        --posLigne;
        ++posColonne;

        //deplacement à droite
        for (unsigned i (0); i < nbPosAInsererDansCeTour; ++i)
            mat[posLigne][posColonne++] = ++valAInser;

        //au augmente le nombre d'insertion dans ce tour
        nbPosAInsererDansCeTour += 2;
    }
}

int main()
{
    cout << "Hello World!" << endl;
    const size_t ordre (101);

    vector <typeCelluleULAM> tabNbPremiers (generNombrePremier(pow(ordre, 2)+1));

    matrice mat (ordre, vector<typeCelluleULAM> (ordre));
    transformeMatEscargotEnUlam (mat, tabNbPremiers);
    genrerMatriceEscargot (mat);
    transformeMatEscargotEnUlam(mat, tabNbPremiers);

    ULAMversPPM(mat);
    ULAMversPPMV2(mat);

    return 0;
}```
