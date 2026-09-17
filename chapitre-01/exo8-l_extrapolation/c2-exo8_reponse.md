#include <iostream>
#include <iomanip>
#include <cmath>

struct Vecteur {
    double x, y, z;
};

struct Quaternion {
    double w, x, y, z;
};

struct Pose {
    Vecteur position;
    Quaternion rotation;
};

Quaternion Multiplier(const Quaternion& a, const Quaternion& b) {
    return {
        a.w*b.w - a.x*b.x - a.y*b.y - a.z*b.z,
        a.w*b.x + a.x*b.w + a.y*b.z - a.z*b.y,
        a.w*b.y - a.x*b.z + a.y*b.w + a.z*b.x,
        a.w*b.z + a.x*b.y - a.y*b.x + a.z*b.w
    };
}

Pose Avancer(const Pose& pose, const Vecteur& vitesseLineaire,
             const Vecteur& vitesseAngulaire, double dt) {
    Pose resultat = pose;

    // Translation à vitesse constante.
    resultat.position.x += vitesseLineaire.x * dt;
    resultat.position.y += vitesseLineaire.y * dt;
    resultat.position.z += vitesseLineaire.z * dt;

    double norme = std::sqrt(
        vitesseAngulaire.x * vitesseAngulaire.x +
        vitesseAngulaire.y * vitesseAngulaire.y +
        vitesseAngulaire.z * vitesseAngulaire.z
    );

    // Cas sans rotation : aucun calcul d'axe ni division par zéro.
    if (norme < 1e-12)
        return resultat;

    double angle = norme * dt;
    double demiAngle = angle * 0.5;
    double s = std::sin(demiAngle) / norme;

    Quaternion delta = {
        std::cos(demiAngle),
        vitesseAngulaire.x * s,
        vitesseAngulaire.y * s,
        vitesseAngulaire.z * s
    };

    resultat.rotation = Multiplier(delta, pose.rotation);

    return resultat;
}

int main() {
    Pose pose;
    Vecteur vitesseLineaire, vitesseAngulaire;
    double dt;

    std::cin >> pose.position.x >> pose.position.y >> pose.position.z;
    std::cin >> pose.rotation.w >> pose.rotation.x
             >> pose.rotation.y >> pose.rotation.z;
    std::cin >> vitesseLineaire.x >> vitesseLineaire.y >> vitesseLineaire.z;
    std::cin >> vitesseAngulaire.x >> vitesseAngulaire.y >> vitesseAngulaire.z;
    std::cin >> dt;

    Pose resultat = Avancer(
        pose, vitesseLineaire, vitesseAngulaire, dt
    );

    std::cout << std::fixed << std::setprecision(4);

    std::cout << resultat.position.x << ' '
              << resultat.position.y << ' '
              << resultat.position.z << '\n';

    std::cout << resultat.rotation.w << ' '
              << resultat.rotation.x << ' '
              << resultat.rotation.y << ' '
              << resultat.rotation.z << '\n';

    return 0;
}