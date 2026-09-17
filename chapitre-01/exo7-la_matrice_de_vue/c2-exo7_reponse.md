#include <iostream>
#include <iomanip>
#include <cmath>

struct Quaternion {
    double w, x, y, z;
};

struct Pose {
    double x, y, z;
    Quaternion q;
};

struct Mat4 {
    double m[4][4];
};

Mat4 PoseToMatrix(const Pose& p) {
    double w=p.q.w, x=p.q.x, y=p.q.y, z=p.q.z;

    return {{
        {1-2*y*y-2*z*z, 2*x*y-2*z*w,   2*x*z+2*y*w,   p.x},
        {2*x*y+2*z*w,   1-2*x*x-2*z*z, 2*y*z-2*x*w,   p.y},
        {2*x*z-2*y*w,   2*y*z+2*x*w,   1-2*x*x-2*y*y, p.z},
        {0,             0,             0,             1}
    }};
}

bool InversionGenerale(const Mat4& a, Mat4& inv) {
    double aug[4][8];

    for (int i=0; i<4; ++i)
        for (int j=0; j<4; ++j) {
            aug[i][j] = a.m[i][j];
            aug[i][j+4] = (i == j) ? 1.0 : 0.0;
        }

    for (int col=0; col<4; ++col) {
        int pivot = col;
        for (int i=col+1; i<4; ++i)
            if (std::abs(aug[i][col]) > std::abs(aug[pivot][col]))
                pivot = i;

        if (std::abs(aug[pivot][col]) < 1e-12)
            return false;

        for (int j=0; j<8; ++j)
            std::swap(aug[col][j], aug[pivot][j]);

        double d = aug[col][col];
        for (int j=0; j<8; ++j)
            aug[col][j] /= d;

        for (int i=0; i<4; ++i) {
            if (i == col) continue;
            double f = aug[i][col];
            for (int j=0; j<8; ++j)
                aug[i][j] -= f * aug[col][j];
        }
    }

    for (int i=0; i<4; ++i)
        for (int j=0; j<4; ++j)
            inv.m[i][j] = aug[i][j+4];

    return true;
}

Mat4 InverseDirecte(const Pose& p) {
    Quaternion c = {p.q.w, -p.q.x, -p.q.y, -p.q.z};

    Mat4 r = PoseToMatrix({
        0, 0, 0, c
    });

    double tx = -p.x, ty = -p.y, tz = -p.z;

    r.m[0][3] = r.m[0][0]*tx + r.m[0][1]*ty + r.m[0][2]*tz;
    r.m[1][3] = r.m[1][0]*tx + r.m[1][1]*ty + r.m[1][2]*tz;
    r.m[2][3] = r.m[2][0]*tx + r.m[2][1]*ty + r.m[2][2]*tz;

    return r;
}

void AfficherEcart(const Mat4& a, const Mat4& b) {
    double maxEcart = 0.0;

    for (int i=0; i<4; ++i)
        for (int j=0; j<4; ++j)
            maxEcart = std::max(maxEcart, std::abs(a.m[i][j] - b.m[i][j]));

    std::cout << std::fixed << std::setprecision(4)
              << "Ecart maximal : " << maxEcart << '\n';
}

void Afficher(const Mat4& m) {
    std::cout << std::fixed << std::setprecision(4);
    for (int i=0; i<4; ++i) {
        for (int j=0; j<4; ++j)
            std::cout << m.m[i][j] << (j == 3 ? '\n' : ' ');
    }
}

int main() {
    Pose pose;
    std::cin >> pose.x >> pose.y >> pose.z;
    std::cin >> pose.q.w >> pose.q.x >> pose.q.y >> pose.q.z;

    Mat4 m = PoseToMatrix(pose);

    Mat4 inverseGenerale;
    bool ok = InversionGenerale(m, inverseGenerale);

    Mat4 inverseDirecte = InverseDirecte(pose);

    if (ok) {
        AfficherEcart(inverseGenerale, inverseDirecte);
        Afficher(inverseGenerale);
        Afficher(inverseDirecte);
    } else {
        std::cout << "Inversion generale : matrice singuliere\n";
        Afficher(inverseDirecte);
    }

    //matrice singulière.
    Mat4 deg = {{
        {0,0,0,0},
        {0,0,0,0},
        {0,0,0,0},
        {0,0,0,1}
    }};

    Mat4 resultat;
    if (!InversionGenerale(deg, resultat))
        std::cout << "Pose dégénérée : inversion impossible\n";

    return 0;
}