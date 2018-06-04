# tp

# install the 'MASS' package before
library(MASS)
# install the 'polynom' package before
library(polynom)

#first set your setwd() to the main directory
#path to the working directory on the SY09 computer : setwd("Z:/Documents/SY09/TP3")
#setwd("Z:/Documents/SY09/TP3")
#path to the working directory on PL computer: setwd("/Users/pierrelouislacorte/Library/Mobile Documents/com~apple~CloudDocs/Documents/5-UTC/3-Cours/SY09/TP3")
setwd("/Users/pierrelouislacorte/Library/Mobile Documents/com~apple~CloudDocs/Documents/5-UTC/3-Cours/SY09/TP3")
#setwd('/Users/matthieu/Google Drive/Cours/GI04/SY09/TP3')
#import function from fonction-tp3 directory
setwd("fonctions-tp3")
source("distXY.R")
source("front.ceuc.R")
source("front.kppv.R")
source("separ1.R")
source("separ2.R")
#import data from donnees-tp3 directory
setwd("..")
#setwd('/Users/matthieu/Google Drive/Cours/GI04/SY09/TP3')
setwd("donnees-tp3")
donn <- read.csv("Synth1-40.csv")
X <- donn[,1:2]
z <- donn[,3]
setwd("..")
#setwd('/Users/matthieu/Google Drive/Cours/GI04/SY09/TP3')

#1.1.1 Classifieur euclidien
ceuc.app <- function(Xapp, zapp)
{
	#calculs des centres de gravite
	#prendre informations sur les classes et les trier par ordre croissant
	classes = sort(unique(zapp))
	#recuperer la longueur des classes (savoir combien de fois on itère sur la variable)
	nb_classes = length(classes)
	#initialisation de mu
	mu = matrix(data = NA, nrow = nb_classes, ncol = ncol(Xapp))
	#pour chaque classe
	for ( i in 1:nb_classes) { 
		#on recupere les individus qui constituent la classe
		individus_de_la_classe = Xapp[which(zapp == classes[i]),]
		#calcul du centre de gravité et ajout à mu
		mu[i,] = (1 / nrow(individus_de_la_classe) * colSums(individus_de_la_classe)) 
	}
	return(mu)
}

ceuc.val <- function(mu, Xtst)
{
	#calcul des distances euclidiennes
	distance = distXY(mu, Xtst)
	#recuperation de la classe la plus proche
	distance_minimum = apply(distance, 2, which.min)
	return(distance_minimum)
}

#1.1.2 K plus proches voisins
kppv.val <- function(Xapp, zapp, K, Xtst)
{
	#calcul des distance entre l'ensemble de test et l'ensemble d'apprentissage
	distanceTestEnsemble = distXY(Xtst,Xapp)
	#ensemble des indices des individus des K plus proches voisins
	Kppv = t(apply(distanceTestEnsemble,1,order)[1:K,])
	#inversement des colonnes et des lignes à ce moment
	#recuperer les classes de ces k plus proches voisins
	df = matrix(zapp[Kppv],ncol = K, nrow = nrow(Xtst))
	# retenir pour chaque individus la classe la plus représentée (attention tendance à tendre vers la première classe du coup)
	v = apply(df,1,function(x) as.numeric(names(which.max(table(x)))))

	return(v)
}

# recuperation du nombre de voisin optimal 
kppv.tune <- function(Xapp, zapp, Xval, zval, nppv)
{
	#initialisation de la valeur optimale de nombre de voisin à la première
	optVoisin = nppv[1]
	#création d'une valeur pour se souvenir du nombre de similarite entre le vecteur de validation et le vecteur deduit
	#on suppose que toutes les valeurs sont donc différentes ici
	nbSimilarite =  0
	for ( i in 1:length(nppv)) { 
		#recuperation des étiquettes estimées à l'aide de nppv[i] voisins
		v = kppv.val (Xapp, zapp, nppv[i], as.matrix(Xval))
		#intersection des étiquettes prédites pour nppv[i] voisins et des étiquettes zval
		#et calcul du nombre de similarité
		sim = length(which(v==zval))
		#si on detecte que le nouveau vecteur a plus de point commun que le precedent alors on met a jour le meilleur
		if(sim > nbSimilarite){
			nbSimilarite = sim
			optVoisin = nppv[i]
		}
	}
	#on retourne le meilleur nombre de voisin
	return(optVoisin)
}

# 1.1.3 Test des fonctions

# pour l'algorithme des classificateurs euclidiens
donn.sep <- separ1(X, z)
Xapp <- donn.sep$Xapp
zapp <- donn.sep$zapp
Xtst <- donn.sep$Xtst
ztst <- donn.sep$ztst

mu <- ceuc.app(Xapp, zapp)
png(file = "res/classificateur_euclidien.png",width=400,height=400)
front.ceuc(mu, Xapp, zapp, 1000)
dev.off()
cat("classificateur_euclidien.png sauvegardee\n")

# pour l'algorithme des kppv
donn.sep <- separ2(X, z)
Xapp <- donn.sep$Xapp
zapp <- donn.sep$zapp
Xval <- donn.sep$Xval
zval <- donn.sep$zval
Xtst <- donn.sep$Xtst
ztst <- donn.sep$ztst
nppv <- 2*(1:6)-1

# détermination du nombre optimal de voisin
Kopt <- kppv.tune(Xapp, zapp, Xval, zval,nppv)
# utilisation de la fonction front.kppv modifiée
png(file = "res/kppv.png",width=400,height=400)
front.kppv(Xapp, zapp, 3, 1000)
dev.off()
cat("kppv.png sauvegardee\n")


# 1.2 Évaluation des performances


# 1.2.1 Jeux de données Synth1-40, Synth1-100, Synth1-500 et Synth1-1000
setwd("donnees-tp3")
donn_Synth1_40 <- read.csv("Synth1-40.csv")
donn_Synth1_100 <- read.csv("Synth1-100.csv")
donn_Synth1_500 <- read.csv("Synth1-500.csv")
donn_Synth1_1000 <- read.csv("Synth1-1000.csv")
name_data = c("Synth1-40","Synth1-100","Synth1-500","Synth1-1000")
setwd("..")
#setwd('/Users/matthieu/Google Drive/Cours/GI04/SY09/TP3')

# 1 ) 
# determination de la proportion des présences des individus des classes

proportion_1_Synth1_40 = (1/40)*length(donn_Synth1_40[,3][donn_Synth1_40[,3]==1])
proportion_2_Synth1_40 = 1 - proportion_1_Synth1_40
paste(c("La proportion des classes pour Synth1-40 est 1 : ", proportion_1_Synth1_40, ", 2 : ", proportion_2_Synth1_40), collapse = " ")

proportion_1_Synth1_100 = (1/100)*length(donn_Synth1_100[,3][donn_Synth1_100[,3]==1])
proportion_2_Synth1_100 = 1 - proportion_1_Synth1_100
paste(c("La proportion des classes pour Synth1-100 est 1 : ", proportion_1_Synth1_100, ", 2 : ", proportion_2_Synth1_100), collapse = " ")

proportion_1_Synth1_500 = (1/500)*length(donn_Synth1_500[,3][donn_Synth1_500[,3]==1])
proportion_2_Synth1_500 = 1 - proportion_1_Synth1_500
paste(c("La proportion des classes pour Synth1-500 est 1 : ", proportion_1_Synth1_500, ", 2 : ", proportion_2_Synth1_500), collapse = " ")

proportion_1_Synth1_1000 = (1/1000)*length(donn_Synth1_1000[,3][donn_Synth1_1000[,3]==1])
proportion_2_Synth1_1000 = 1 - proportion_1_Synth1_1000
paste(c("La proportion des classes pour Synth1-1000 est 1 : ", proportion_1_Synth1_1000, ", 2 : ", proportion_2_Synth1_1000), collapse = " ")

# calculs des centres de gravites

# fonction pour calculer les centres de gravité
centre_gravite_par_classe <- function(X)
{
	#calculs des centres de gravite
	#prendre informations sur les classes et les trier par ordre croissant
	classes = sort(unique(X[,3]))
	#recuperer la longueur des classes (savoir combien de fois on itère sur la variable)
	nb_classes = length(classes)
	#initialisation de mu
	mu = matrix(data = NA, nrow = nb_classes, ncol = ncol(X)-1)
	#pour chaque classe
	for ( i in 1:nb_classes) { 
		#on recupere les individus qui constituent la classe
		individus_de_la_classe = X[which(X[,3] == classes[i]),]
		#calcul du centre de gravité et ajout à mu
		mu[i,] = (1 / nrow(individus_de_la_classe) * colSums(individus_de_la_classe[,1:2])) 
	}
	rownames(mu) <- c("classe 1", "classe 2")
	colnames(mu) <- c("V1","V2")
	return(mu)
}
# recuperation des centres de gratives
centre_gravite_Synth1_40 = centre_gravite_par_classe(donn_Synth1_40)
centre_gravite_Synth1_100 = centre_gravite_par_classe(donn_Synth1_100)
centre_gravite_Synth1_500 = centre_gravite_par_classe(donn_Synth1_500)
centre_gravite_Synth1_1000 = centre_gravite_par_classe(donn_Synth1_1000)

# afficher resultat
centre_gravite_Synth1_40
centre_gravite_Synth1_100
centre_gravite_Synth1_500
centre_gravite_Synth1_1000

# fonction pour calculer les covariances par classes

covariance_par_classe <- function(X)
{
	#calculs des centres de gravite
	#prendre informations sur les classes et les trier par ordre croissant
	classes = sort(unique(X[,3]))
	#recuperer la longueur des classes (savoir combien de fois on itère sur la variable)
	nb_classes = length(classes)
	#initialisation de mu
	mu = list();
	#pour chaque classe
	for ( i in 1:nb_classes) { 
		#on recupere les individus qui constituent la classe
		individus_de_la_classe = X[which(X[,3] == classes[i]),]
		#calcul de la covariance et ajout à mu
		mu[[i]]= cov(individus_de_la_classe[,1:2])
	}
	names(mu) <- paste(c("cov classe", "cov classe"), 1:2, sep = "") 
	mu
}

# recuperation des covariances
covariance_Synth1_40 = covariance_par_classe(donn_Synth1_40)
covariance_Synth1_100 = covariance_par_classe(donn_Synth1_100)
covariance_Synth1_500 = covariance_par_classe(donn_Synth1_500)
covariance_Synth1_1000 = covariance_par_classe(donn_Synth1_1000)

# afficher resultat
covariance_Synth1_40
covariance_Synth1_100
covariance_Synth1_500
covariance_Synth1_1000

# 2 )

calcul_tx_erreurs_euc <- function(X, N,nbcol){
    
    # initialisation d'une matrice pour récupérer les tx_erreurs_calculés
    m = matrix(data = NA, nrow = 20, ncol = 2)
    for ( i in 1:N) {
        # definition d'un nouveau jeu de données à partir du jeu fournis
        # utilisation plusieurs fois de separ1 pour ne pas que Xapp, Xval et Xtst dépendent les uns des autres
        donn.sep <- separ1(X[,1:nbcol], X[,nbcol+1])
        Xapp <- donn.sep$Xapp
        zapp <- donn.sep$zapp
        donn.sep <- separ1(X[,1:nbcol], X[,nbcol+1])
        Xtst <- donn.sep$Xtst
        ztst <- donn.sep$ztst
        
        # recuperation de la matrice d'apprentissage
        mApp = ceuc.app(Xapp,zapp)
        # recupération des vecteurs déduit
        # sur l'apprentissage
        vApp = ceuc.val(mApp,Xapp)
        # sur le test
        vTst = ceuc.val(mApp,Xtst)
        # calcul du taux d'erreur sur l'ensemble d'apprentissage
        simApp = length(which(vApp==zapp))
        tx_erreur_app = 1 - (1/nrow(Xapp)) * simApp
        # calcul du taux d'erreur sur l'ensemble de test
        simTst = length(which(vTst==ztst))
        tx_erreur_tst = 1 - (1/nrow(Xtst)) * simTst
        # sauvegarde des taux d'erreurs
        m[i,] <- c(tx_erreur_app,tx_erreur_tst)
        colnames(m) <- c("tx_erreur_app","tx_erreur_tst")
    }
    
    # estimation ponctuelle
    estimation_ponctuelle = (colSums(m)/N)
    min = c(t.test(m[,1])$conf.int[1],t.test(m[,2])$conf.int[1])
    max = c(t.test(m[,1])$conf.int[2],t.test(m[,2])$conf.int[2])
    #colnames(min_intervalle)<-c("conf95_app","conf95_tst")
    
    return(cbind(estimation_ponctuelle,min,max)* 100)
}

# test de la fonction sur les jeux données
tx_erreur_Synth1_40_euc = calcul_tx_erreurs_euc(donn_Synth1_40, 20, 2)
tx_erreur_Synth1_100_euc = calcul_tx_erreurs_euc(donn_Synth1_100, 20, 2)
tx_erreur_Synth1_500_euc = calcul_tx_erreurs_euc(donn_Synth1_500, 20, 2)
tx_erreur_Synth1_1000_euc = calcul_tx_erreurs_euc(donn_Synth1_1000, 20, 2)

# afficher resultats
tx_erreur_Synth1_40_euc
tx_erreur_Synth1_100_euc
tx_erreur_Synth1_500_euc
tx_erreur_Synth1_1000_euc

# 3 ) calcul du nombre optimal de voisin pour les jeux de données
Kopt_Synth1_40 <- kppv.tune (donn_Synth1_40[,1:2], donn_Synth1_40[,3], donn_Synth1_40[,1:2], donn_Synth1_40[,3], 2*(1:6)-1)
Kopt_Synth1_100 <- kppv.tune (donn_Synth1_100[,1:2], donn_Synth1_100[,3], donn_Synth1_100[,1:2], donn_Synth1_100[,3], 2*(1:6)-1)
Kopt_Synth1_500 <- kppv.tune (donn_Synth1_500[,1:2], donn_Synth1_500[,3], donn_Synth1_500[,1:2], donn_Synth1_500[,3], 2*(1:6)-1)
Kopt_Synth1_1000 <- kppv.tune (donn_Synth1_1000[,1:2], donn_Synth1_1000[,3], donn_Synth1_1000[,1:2], donn_Synth1_1000[,3], 2*(1:6)-1)


# 4 ) 

calcul_tx_erreurs_kppv <- function(X, N, nbcol){

	# initialisation d'une matrice pour récupérer les tx_erreurs_calculés
	m = matrix(data = NA, nrow = 20, ncol = 2)
	nppv <- 2*(1:6)-1

	for ( i in 1:N) { 
		# definition d'un nouveau jeu de données à partir du jeu fournis
		# utilisation plusieurs fois de separ2 pour ne pas que Xapp, Xval et Xtst dépendent les uns des autres
		donn.sep <- separ2(X[,1:nbcol], X[,nbcol+1])
		Xapp <- donn.sep$Xapp
		zapp <- donn.sep$zapp
		donn.sep <- separ2(X[,1:nbcol], X[,nbcol+1])
		Xval <- donn.sep$Xval
		zval <- donn.sep$zval
		donn.sep <- separ2(X[,1:nbcol], X[,nbcol+1])
		Xtst <- donn.sep$Xtst
		ztst <- donn.sep$ztst

		# recuperation du nombre de voisins optimals
		Kopt <- kppv.tune(Xapp, zapp, Xval, zval, nppv)
		# recupération des vecteurs déduit 
		# sur l'apprentissage
		vApp = kppv.val(Xapp, zapp, Kopt, Xapp)
		# sur le test
		vTst = kppv.val(Xapp, zapp, Kopt, Xtst)
		# calcul du taux d'erreur sur l'ensemble d'apprentissage
		simApp = length(which(vApp==zapp))
		tx_erreur_app = 1 - (1/nrow(Xapp)) * simApp
		# calcul du taux d'erreur sur l'ensemble de test
		simTst = length(which(vTst==ztst))
		tx_erreur_tst = 1 - (1/nrow(Xtst)) * simTst
		# sauvegarde des taux d'erreurs
		m[i,] <- c(tx_erreur_app,tx_erreur_tst)
		colnames(m) <- c("tx_erreur_app","tx_erreur_tst")
	}	

	# estimation ponctuelle
	estimation_ponctuelle = (colSums(m)/N)
    min = c(max(t.test(m[,1])$conf.int[1],0),max(t.test(m[,2])$conf.int[1],0))
    max = c(t.test(m[,1])$conf.int[2],t.test(m[,2])$conf.int[2])
    
	return(cbind(estimation_ponctuelle,min,max)*100)
}

# test de la fonction sur les jeux données
tx_erreur_Synth1_40_kppv = calcul_tx_erreurs_kppv(donn_Synth1_40, 20, 2)
tx_erreur_Synth1_100_kppv = calcul_tx_erreurs_kppv(donn_Synth1_100, 20, 2)
tx_erreur_Synth1_500_kppv = calcul_tx_erreurs_kppv(donn_Synth1_500, 20, 2)
tx_erreur_Synth1_1000_kppv = calcul_tx_erreurs_kppv(donn_Synth1_1000, 20, 2)

# afficher resultats
tx_erreur_Synth1_40_kppv
tx_erreur_Synth1_100_kppv
tx_erreur_Synth1_500_kppv
tx_erreur_Synth1_1000_kppv

# représentation graphique des taux d'erreurs d'apprentissage sur kppv
png(file = "res/tx_erreur_app_kppv.png",width=400,height=400)
v_tx_erreur_app_kppv = c(tx_erreur_Synth1_40_kppv[1], tx_erreur_Synth1_100_kppv[1], tx_erreur_Synth1_500_kppv[1], tx_erreur_Synth1_1000_kppv[1])
#barplot(v_tx_erreur_kppv,main = "Representation du taux d'erreur d'apprentissage Kppv", ylab="Taux d'erreur", xaxt="n",xlab = "Jeux de donnees")
#axis(1,at=1:4, labels=name_data)
dev.off()
cat("tx_erreur_app_kppv.png sauvegardee\n")

# représentation graphique des taux d'erreurs de test sur kppv
png(file = "res/tx_erreur_tst_kppv.png",width=400,height=400)
v_tx_erreur_tst_kppv = c(tx_erreur_Synth1_40_kppv[2], tx_erreur_Synth1_100_kppv[2], tx_erreur_Synth1_500_kppv[2], tx_erreur_Synth1_1000_kppv[2])
#barplot(v_tx_erreur_tst_kppv,main = "Representation du taux d'erreur de test Kppv", ylab="Taux d'erreur", xaxt="n",xlab = "Jeux de donnees")
#axis(1,at=1:4, labels=name_data)
dev.off()
cat("tx_erreur_tst_kppv.png sauvegardee\n")

# 1.2.1 Jeu de données Synth2-1000
# on recupere les données 
setwd("donnees-tp3")
donn_Synth2_1000 <- read.csv("Synth2-1000.csv")
setwd("..")

# calcul des proportions des individus appartenant à chaque classe
proportion_1_Synth2_1000 = (1/1000)*length(donn_Synth2_1000[,3][donn_Synth2_1000[,3]==1])
proportion_2_Synth2_1000 = 1 - proportion_2_Synth1_1000
paste(c("La proportion des classes pour Synth2-1000 est 1 : ", proportion_1_Synth2_1000, ", 2 : ", proportion_2_Synth2_1000), collapse = " ")

# recuperation des centres de gravite des classes
centre_gravite_Synth2_1000 = centre_gravite_par_classe(donn_Synth2_1000)

# afficher resultat
centre_gravite_Synth2_1000

# calcul des covariances 
covariance_Synth2_1000 = covariance_par_classe(donn_Synth2_1000)

# afficher resultat
covariance_Synth2_1000

# calcul du taux d'erreur ponctuel euclidien 
tx_erreur_Synth2_1000_euc = calcul_tx_erreurs_euc(donn_Synth2_1000, 20 ,2)

# affichage de ce taux d'erreur
tx_erreur_Synth2_1000_euc

# calcul du taux d'erreur ponctuel kppv
tx_erreur_Synth2_1000_kppv = calcul_tx_erreurs_kppv(donn_Synth2_1000, 20 ,2)

# affichage de ce taux d'erreur
tx_erreur_Synth2_1000_kppv

# 1.2.3 Jeux de données réelles
# Données Pima et Breastcancer
setwd("donnees-tp3")
donn_Pima <- read.csv("Pima.csv")
donn_Breastcancer <- read.csv("Breastcancer.csv")
setwd('..')

# test de la fonction euc sur les jeux données
tx_erreur_Pima_euc = calcul_tx_erreurs_euc(donn_Pima, 20, 7)
tx_erreur_Breastcancer_euc = calcul_tx_erreurs_euc(donn_Pima, 20, 7)


# afficher resultats
tx_erreur_Pima_euc
tx_erreur_Breastcancer_euc

# test de la fonction kppv sur les jeux données
tx_erreur_Pima_kppv = calcul_tx_erreurs_kppv(donn_Pima, 20, 7)
tx_erreur_Breastcancer_kppv = calcul_tx_erreurs_kppv(donn_Pima, 20, 7)


# afficher resultats
tx_erreur_Pima_kppv
tx_erreur_Breastcancer_kppv

# frontiere de décision Synth1-n
png(file = "res/frontiere_decision_synth1-100.png",width=500,height=500)
plot(donn_Synth1_100[,1],donn_Synth1_100[,2],col=c("blue","orange")[donn_Synth1_100[,3]],ylab="V2",xlab="V1")
abline(a=-3/2, b=-2)
dev.off()
cat("frontiere_decision_synth1-100.png sauvegardee\n")

png(file = "res/frontiere_decision_synth1-1000.png",width=500,height=500)
plot(donn_Synth1_1000[,1],donn_Synth1_1000[,2],col=c("blue","orange")[donn_Synth1_1000[,3]],ylab="V2",xlab="V1")
abline(a=-3/2, b=-2)
dev.off()
cat("frontiere_decision_synth1-1000.png sauvegardee\n")

# frontiere de décision Synth2-1000
png(file = "res/frontiere_decision_synth2-1000.png",width=500,height=500)
plot(donn_Synth2_1000[,1],donn_Synth2_1000[,2],col=c("blue","orange")[donn_Synth2_1000[,3]],ylab="V2",xlab="V1")
lines(polynomial(c(5*log(5) - 41, 34,-4)),len=2000)
dev.off()
cat("frontiere_decision_synth2-1000.png sauvegardee\n")


# sauvegarde de la représentation des jeux de données Synth1-n en fonction de leur appartenance a telle ou telle classe
png(file = "res/representation_pop_synth1-40.png",width=500,height=500)
plot(donn_Synth1_40[,1],donn_Synth1_40[,2],col=c("blue","orange")[donn_Synth1_40[,3]],ylab="V2",xlab="V1")
dev.off()
cat("representation_pop_synth1-40.png sauvegardee\n")

png(file = "res/representation_pop_synth1-100.png",width=500,height=500)
plot(donn_Synth1_100[,1],donn_Synth1_100[,2],col=c("blue","orange")[donn_Synth1_100[,3]],ylab="V2",xlab="V1")
dev.off()
cat("representation_pop_synth1-100.png sauvegardee\n")

png(file = "res/representation_pop_synth1-500.png",width=500,height=500)
plot(donn_Synth1_500[,1],donn_Synth1_500[,2],col=c("blue","orange")[donn_Synth1_500[,3]],ylab="V2",xlab="V1")
dev.off()
cat("representation_pop_synth1-500.png sauvegardee\n")

png(file = "res/representation_pop_synth1-1000.png",width=500,height=500)
plot(donn_Synth1_1000[,1],donn_Synth1_1000[,2],col=c("blue","orange")[donn_Synth1_1000[,3]],ylab="V2",xlab="V1")
dev.off()
cat("representation_pop_synth1-1000.png sauvegardee\n")

png(file = "res/representation_pop_synth2-1000.png",width=500,height=500)
plot(donn_Synth2_1000[,1],donn_Synth2_1000[,2],col=c("blue","orange")[donn_Synth2_1000[,3]],ylab="V2",xlab="V1")
dev.off()
cat("representation_pop_synth2-1000.png sauvegardee\n")
