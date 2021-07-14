# BIN

#include <iostream>
#include<cmath>
#include <fstream>
#include <string>
#include <ctime>
#include <iomanip>
using namespace std;
class TTarget {
public:
	double x0 = 0, y0 = 0, xt = 0, yt = 0;
	double V = 0;
	double K = 0;
	enum  Tipceli { TAircraft, TMissile };
	double t0 = 0;
	double t = 0;
	virtual void Move(double t) = 0;
	double GET_X() { return xt; };
	double GET_Y() { return yt; };
	TTarget() {};

};
class TAircraft :public TTarget {
public:
	void Move(double t) 
	{
		cout << "Aircraft: ";
		xt = (x0 - V * cos(K) * (t - t0));
		cout << xt;
		yt = (y0 - V * sin(K) * (t - t0));
		cout << " " << yt<<endl<<endl;
		GET_X();
		GET_Y();

	}
};
class TMissile : public TTarget {
private:
public: double N;
	  double _N() 
	  {
		  N = 2;
		  return N;
	  }
	  void Move(double t) 
	  {
		  cout << "Missile: ";
		  xt = (x0 - ((V + _N() * (t - t0)) * cos(K) * (t - t0)));
		  yt = (y0 - ((V + _N() * (t - t0)) * sin(K) * (t - t0)));
		   cout << xt << " " << yt << endl << endl;
		  GET_X();
		  GET_Y();
	  }
	  TMissile() { };
	  ~TMissile() {};
};
class RLS {
public:
	double Xrls = 0, Yrls = 0;
	int tk = 0, t0 = 0, z = 0, dt = 0;
	double D = 0;
	long double Ro = 0;
	int  n1 = 0, n2 = 0, n = 0, Kolvo = 0;
	void Peleng();
	TTarget** massiv = new TTarget * [10];
	RLS() {};
	~RLS() {};
};
void RLS::Peleng() {
	ofstream newfiles;
	srand(time(0));
	newfiles.open("newtextfile4.txt");
	z = rand() % 2;
	newfiles << "x RLS:";
	if (z == 0) {
		Xrls = rand() % 10;
	}
	if (z == 1) {
		Xrls = rand() % 10 * -1;
	}
	newfiles << Xrls << ' ';
	newfiles << "y RLS:";
	Yrls = rand() % 10;
	newfiles << Yrls << ' ';
	newfiles << " R: ";
	Ro = 1000;
	newfiles << Ro << endl;
	newfiles << "start t:";
	t0 = 0;
	newfiles << t0 << ' ';
	newfiles << "End t:";
	tk = rand() % 10 + 5;
	newfiles << tk << ' ';
	newfiles << "Time step:";
	dt = 1 + rand() % 3;
	newfiles << dt << endl;
	newfiles << endl;
	double sumsredn = 0;
	double srednee = 0;
	srand(time(0));
	double d1 = 0, d2 = 0;
	D = 0;
	double AZ = 0;
	double t;
	for (int i = 0; i < 10; i++)
	{
		double srAz = 0;
		double srAz2 = 0;
		int m = 0, n = 0, r = 0, p = 0;;
		double u = rand() % 2;
		t = t0;
		if (u == 0) {
			newfiles << "Assumption...Aircraft/Missle?" << endl;
			massiv[i] = new TAircraft;
			r = rand() % 2;
			p = rand() % 2;
			if (r == 0) {
				massiv[i]->x0 = (rand() % 100 + 1);
			}
			if (r == 1) {
				massiv[i]->x0 = (rand() % 100 + 1) * -1;
			}
			newfiles << "x0=" << massiv[i]->x0 << ' ';
			if (p == 0) {
				massiv[i]->y0 = (rand() % 100 + 1);
			}
			if (p == 1) {
				massiv[i]->y0 = (rand() % 100 + 1) * -1;
			}
			newfiles << "y0=" << massiv[i]->y0 << ' ';
			massiv[i]->V = (rand() % 10 + 20);
			newfiles << "V=" << massiv[i]->V << ' ';
			if ((massiv[i]->x0 > 0) && (massiv[i]->y0 > 0)) {
				massiv[i]->K = (rand() % 90 + 91);
			}
			if ((massiv[i]->x0 < 0) && (massiv[i]->y0 > 0)) {
				massiv[i]->K = (rand() % 90 + 1);
			}
			if ((massiv[i]->x0 > 0) && (massiv[i]->y0 < 0)) {
				massiv[i]->K = (rand() % 90 + 181);
			}
			if ((massiv[i]->x0 < 0) && (massiv[i]->y0 < 0)) {
				massiv[i]->K = (rand() % 90 + 271);
			}
			newfiles << "K=" << massiv[i]->K << endl;
			newfiles << "Calculation..." << endl;
			newfiles << "-----------------------------------------" << endl;
			newfiles << " Visibility    |time " << "  " << "|   D   " << " " << "|  AZ   |" << endl;
			newfiles << "-----------------------------------------" << endl;
			for (t; t < tk; t += dt) {
				cout << "time: " << t << endl;
				massiv[i]->t0 = t0;
				massiv[i]->t = t;
				massiv[i]->Move(t);
				d1 = pow(massiv[i]->GET_X() - Xrls, 2);
				d2 = pow(massiv[i]->GET_Y() - Yrls, 2);
				D = sqrt(d1 + d2);
				if (D <= Ro) {
					newfiles << "Target detected";
					if ((massiv[i]->GET_Y() > 0) && (massiv[i]->GET_X() < 0)) {
						AZ = abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14;
					}
					if ((massiv[i]->GET_Y() > 0) && (massiv[i]->GET_X() > 0)) {
						AZ = abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14 + 90;
					}
					if ((massiv[i]->GET_Y() < 0) && (massiv[i]->GET_X() > 0)) {
						AZ = (abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14) + 180;
					}
					if ((massiv[i]->GET_Y() < 0) && (massiv[i]->GET_X() < 0)) {
						AZ = (abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14) + 270;
					}
					newfiles << fixed << showpoint;
					newfiles << setprecision(2);
					newfiles << "| " << t << "  |" << " " << "" << D << " |" << " " << AZ << "|" << endl;
					newfiles << "-----------------------------------------" << endl;
					srAz += AZ;
					m++;
				}
				else if (D > Ro) {
					newfiles << "Not detected   ";
					newfiles << fixed << showpoint;
					newfiles << setprecision(2);
					newfiles << "| " << t << "  |" << endl;
					newfiles << "-----------------------------------------" << endl;
				}
				if (D == 0) {
					newfiles << "Target destroy!!";
				}
			}
			if (D <= Ro)
				newfiles << "->Its Aircraft------>" << "Average value AZ:" << srAz / m << endl;
		}

		if (u == 1) {
			newfiles << "Assumption...Missle/Aircraft?" << endl;
			massiv[i] = new TMissile;
			TMissile uskorenie;
			uskorenie._N();
			newfiles << "N=" << uskorenie._N() << ' ';
			r = rand() % 2;
			p = rand() % 2;
			if (r == 0) {
				massiv[i]->x0 = (rand() % 100 + 1);
			}
			if (r == 1) {
				massiv[i]->x0 = (rand() % 100 + 1) * -1;
			}
			newfiles << "x0=" << massiv[i]->x0 << ' ';
			if (p == 0) {
				massiv[i]->y0 = (rand() % 100 + 1);
			}
			if (p == 1) {
				massiv[i]->y0 = (rand() % 100 + 1) * -1;
			}
			massiv[i]->y0 = rand() % 1000 + 1;
			newfiles << "y0=" << massiv[i]->y0 << ' ';
			massiv[i]->V = rand() % 10 + 1;
			newfiles << "V=" << massiv[i]->V << ' ';
			if ((massiv[i]->x0 > 0) && (massiv[i]->y0 > 0)) {
				massiv[i]->K = (rand() % 90 + 91);
			}
			if ((massiv[i]->x0 < 0) && (massiv[i]->y0 > 0)) {
				massiv[i]->K = (rand() % 90 + 1);
			}
			if ((massiv[i]->x0 > 0) && (massiv[i]->y0 < 0)) {
				massiv[i]->K = (rand() % 90 + 181);
			}
			if ((massiv[i]->x0 < 0) && (massiv[i]->y0 < 0)) {
				massiv[i]->K = (rand() % 90 + 271);
			}
			newfiles << "K=" << massiv[i]->K << endl;
			newfiles << "Calculation..." << endl;
			newfiles << "-----------------------------------------" << endl;
			newfiles << " Visibility    |time " << "  " << "|   D   " << " " << "|  AZ   |" << endl;
			newfiles << "-----------------------------------------" << endl;
			for (t; t < tk; t += dt) {
				massiv[i]->t0 = t0;
				massiv[i]->t = t;
				massiv[i]->Move(t);
				d1 = pow(massiv[i]->GET_X() - Xrls, 2);
				d2 = pow(massiv[i]->GET_Y() - Yrls, 2);
				D = sqrt(d1 + d2);
				if (D <= Ro) {
					newfiles << "Target detected";
					if ((massiv[i]->GET_Y() > 0) && (massiv[i]->GET_X() < 0)) {
						AZ = abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14;
					}
					if ((massiv[i]->GET_Y() > 0) && (massiv[i]->GET_X() > 0)) {
						AZ = abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14 + 90;
					}
					if ((massiv[i]->GET_Y() < 0) && (massiv[i]->GET_X() > 0)) {
						AZ = (abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14) + 180;
					}
					if ((massiv[i]->GET_Y() < 0) && (massiv[i]->GET_X() < 0)) {
						AZ = (abs(atan((massiv[i]->GET_Y() - Yrls) / (massiv[i]->GET_X() - Xrls))) * 180 / 3.14) + 270;
					}
					newfiles << fixed << showpoint;
					newfiles << setprecision(2);
					newfiles << "| " << t << "  |" << " " << "" << D << " |" << " " << AZ << "|" << endl;
					newfiles << "-----------------------------------------" << endl;
					srAz2 += AZ;
					n++;
				}
				else if (D > Ro) {
					newfiles << "Not detected   ";
					newfiles << fixed << showpoint;
					newfiles << setprecision(2);
					newfiles << "| " << t << "  |" << endl;
					newfiles << "-----------------------------------------" << endl;
				}
				if (D == 0) {
					newfiles << "Target destroy!!";
				}
			}
			if (D <= Ro)
				newfiles << "->Its Missle------>" << "Average value AZ:" << srAz2 / n << endl;
		}
		newfiles << endl;
	}
	delete[] * massiv;
}
int main()
{
	time(NULL);
	cout << "Launched at " << __TIMESTAMP__<<endl;
	RLS novayacel;
	novayacel.RLS::Peleng();
	cout << "Landed at " << __TIME__;
}
