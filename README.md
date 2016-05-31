# projet-android-
package p8.demo.p8sokoban;

import android.content.Context;
import android.content.res.Resources;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.util.Log;
import android.view.KeyEvent;
import android.view.MotionEvent;
import android.view.SurfaceHolder;
import android.view.SurfaceView;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import java.util.Random;

public class SokobanView extends SurfaceView implements SurfaceHolder.Callback, Runnable {

    // Declaration des images


    // constante modelisant les differentes types de cases
    static final int CST_block = 0;
    static final int CST_diamant = 1;
    static final int CST_perso = 2;
    static final int CST_zone = 3;

   //  les variable couleur
    private Bitmap vide;

    // initialisation des couleur de grand taille
    private Bitmap blanc_max;
    private Bitmap rouge_max;
    private Bitmap jaune_max;
    private Bitmap mauve_max;
    private Bitmap blue_max;
    private Bitmap vert_max;

    private Bitmap blanc_min;
    private Bitmap rouge_min;
    private Bitmap jaune_min;
    private Bitmap mauve_min;
    private Bitmap blue_min;
    private Bitmap vert_min;


    // Declaration des objets Ressources et Context permettant d'accï¿½der aux ressources de notre application et de les charger
    private Resources mRes;
    private Context mContext;

    // tableau modelisant la carte du jeu
    int[][] carte;

    // ancres pour pouvoir centrer la carte du jeu
    int carteTopAnchor;                   // coordonnï¿½es en Y du point d'ancrage de notre carte
    int carteLeftAnchor;                  // coordonnï¿½es en X du point d'ancrage de notre carte

    // variable de lemplacement initialiser aune place indifiner

    int place=3; // la place de chaque  vecteur
    int pv;
    boolean clic=false; //teste de touche sur la zone des vecteur

    // taille de la carte
    static final int carteWidth = 8;
    static final int carteHeight = 8;
    static final int carteTileSize = 40;
    static final int carteTileSizemin = 35;

    // constante modelisant les differentes types de cases

    static final int CST_mauve = 6;
    static final int CST_vert = 5;
    static final int CST_blue = 4;
    static final int CST_blanc = 3;
    static final int CST_rouge = 2;
    static final int CST_jaune = 1;
    static final int CST_vide = 0;

    int[][] vect;
    int[][] sup; // vecteur utiliser a la suprission des case de meme couleur
    int[] depX={36,141,247};
    int[] depY={330,330,330};

    // tableau de reference du terrain
    int[][] ref1 = {
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide},
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide},
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide},
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide},
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide},
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide},
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide},
            {CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide, CST_vide}

    };


    // position de reference des diamants
    int[][] refdiamants = {
            {2, 3},
            {2, 6},
            {6, 3},
            {6, 6}
    };

    // position de reference du joueur
    int refxPlayer = 4;
    int refyPlayer = 1;

    // position courante des diamants
    int[] position = {0, 0, 0};


    // position courante du joueur
    int xPlayer = 4;
    int yPlayer = 1;

    /* compteur et max pour animer les zones d'arrivï¿½e des diamants */
    int currentStepZone = 0;
    int maxStepZone = 4;

    // thread utiliser pour animer les zones de depot des diamants
    private boolean in = true;
    private Thread cv_thread;
    SurfaceHolder holder;

    Paint paint;

    /**
     * The constructor called from the main JetBoy activity
     *
     * @param context
     * @param attrs
     */
    public SokobanView(Context context, AttributeSet attrs) {
        super(context, attrs);


        // permet d'ecouter les surfaceChanged, surfaceCreated, surfaceDestroyed
        holder = getHolder();
        holder.addCallback(this);

        // chargement des images
        mContext = context;
        mRes = mContext.getResources();
        vide = BitmapFactory.decodeResource(mRes, R.drawable.apture1);
        blue_max = BitmapFactory.decodeResource(mRes, R.drawable.blue);
        rouge_max = BitmapFactory.decodeResource(mRes, R.drawable.rouge);
        jaune_max = BitmapFactory.decodeResource(mRes, R.drawable.jeune);
        vert_max = BitmapFactory.decodeResource(mRes, R.drawable.vert);
        blanc_max = BitmapFactory.decodeResource(mRes, R.drawable.blanc);
        mauve_max = BitmapFactory.decodeResource(mRes, R.drawable.mauve);

        // couleur de petit carre
        blue_min = BitmapFactory.decodeResource(mRes, R.drawable.blue_min);
        rouge_min = BitmapFactory.decodeResource(mRes, R.drawable.rouge_min);
        jaune_min = BitmapFactory.decodeResource(mRes, R.drawable.jeune_min);
        vert_min = BitmapFactory.decodeResource(mRes, R.drawable.vert_min);
        blanc_min = BitmapFactory.decodeResource(mRes, R.drawable.blanc_min);
        mauve_min = BitmapFactory.decodeResource(mRes, R.drawable.mauve_min);

                // initialisation des parmametres du jeu
        initparameters();

        // creation du thread
        cv_thread = new Thread(this);
        // prise de focus pour gestion des touches
        setFocusable(true);

    }

    // chargement du niveau a partir du tableau de reference du niveau
    private void loadlevel() {
        for (int i = 0; i < carteHeight; i++) {
            for (int j = 0; j < carteWidth; j++) {

                carte[j][i] = ref1[j][i];


            }
        }
    }

    private void loadvect() {
        Random generateur = new Random();
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                int randomint = 1 + generateur.nextInt(7 - 1);
                vect[i][j] = randomint;

            }
        }

    }

    // initialisation du jeu
    public void initparameters() {

    	/*
    	paint = new Paint();
    	paint.setColor(0xff0000);

    	paint.setDither(true);
    	paint.setColor(0xffff6600);
    	paint.setStyle(Paint.Style.STROKE);
    	paint.setStrokeJoin(Paint.Join.ROUND);
    	paint.setStrokeCap(Paint.Cap.ROUND);
    	paint.setStrokeWidth(3);
    	paint.setTextAlign(Paint.Align.LEFT);
    	*/

        carte = new int[carteHeight][carteWidth];
        loadlevel();
        vect = new int[3][3];
        loadvect();
        carteTopAnchor = (getHeight() - carteHeight * carteTileSize) / 2;
        carteLeftAnchor = (getWidth() - carteWidth * carteTileSize) / 2;
        xPlayer = refxPlayer;
        yPlayer = refyPlayer;

        if ((cv_thread != null) && (!cv_thread.isAlive())) {
            cv_thread.start();
            Log.e("-FCT-", "cv_thread.start()");
        }
    }

    // dessin des fleches
   /* private void paintarrow(Canvas canvas) {


    	canvas.drawBitmap(up, (getWidth()-up.getWidth())/2, 0, null);
    	canvas.drawBitmap(down, (getWidth()-down.getWidth())/2, getHeight()-down.getHeight(), null);
    	canvas.drawBitmap(left, 0, (getHeight()-up.getHeight())/2, null);
    	canvas.drawBitmap(right, getWidth()-right.getWidth(), (getHeight()-up.getHeight())/2, null);
    }*/

    // foction qui returne la position
    int Position(int p) {
        if (p < 3)
            p++;
        else p = 0;

        return p;
    }

    // fonction qui suprime des cases de meme couleur

    public void suprimme(){

        sup=new int [2][20];
        int h=0;
        int trouve=0;
        int cpt=0;
        int valeur=0;
        for (int i=0;i<carteWidth;i++){
            cpt=0;
            for (int j=0;j<carteHeight;j++){
                if(carte[i][j]==valeur && valeur!=0){
                    cpt++;
                }else{
                    valeur=carte[i][j];
                    cpt=0;
                }switch(cpt){
                        case 2: sup[0][h]=i;sup[1][h]=j-2;h++;
                                sup[0][h]=i;sup[1][h]=j-1;h++;
                                sup[0][h]=i;sup[1][h]=j;h++;
                                break;
                        case 3: sup[0][h]=i;sup[1][h]=j;h++;
                        case 4: sup[0][h]=i;sup[1][h]=j;h++;
                }
            }}
        for (int i=0;i<carteWidth;i++){
            cpt=0;
            for (int j=0;j<carteHeight;j++){
                if(carte[j][i]==valeur && valeur!=0){
                    cpt++;
                }else{
                    valeur=carte[j][i];
                    cpt=0;
                }switch(cpt){ //recupirer les indices de cases a suprimmer
                    case 2: sup[0][h]=j-2;sup[1][h]=i;h++;
                        sup[0][h]=j-1;sup[1][h]=i;h++;
                        sup[0][h]=j;sup[1][h]=i;h++;
                        break;
                    case 3: sup[0][h]=j;sup[1][h]=i;h++;
                    case 4: sup[0][h]=j;sup[1][h]=i;h++;
                }
            }}
          for (int ks=0;ks<h;ks++){

              carte[sup[0][ks]][sup[1][ks]]=0;
          }
    }


    // la fonction qui ne charge des nouveau vecteur aleatoirement

    public void Changevect(int p){

        Random generateur = new Random();
            for (int j = 0; j < 3; j++) {
                int randomint = 1 + generateur.nextInt(7 - 1);
                vect[p][j] = randomint;

            }
    }

    // fonction qui teste c'est les case de la matrice est vide

    public boolean Estvide(int x, int y,int pos){
        int i= x/ 40;
        int j=y/40;
        int vide=0;
        boolean estvide=false;
        switch(pos) {
            case 0 :
                int L = j - 3;
                if (y < 350 && y > 125) {

                    for (int k = L; k < j; k++) {
                        if (carte[k][i] == 0) vide++;
                    }

                } break;
            case 2 :
                int L2 = j - 3;
                if (y < 350 && y > 125) {

                    for (int k = L2; k < j; k++) {
                        if (carte[k][i] == 0) vide++;
                    }

                } break;
            case 1 : int L1 = i - 1;
                int j1=j-2;
                if (y < 400 && y > 70 && x>40 && x<260) {

                    for (int k = L1; k < i+2; k++) {
                        if(carte[j1][k] ==0) vide++ ;


                    }
                } break;
            case 3 : int L3 = i - 1;
                int j4=j-2;
                if (y < 400 && y > 70 && x>40 && x<260) {

                    for (int k = L3; k < i+2; k++) {
                        if(carte[j4][k]==0) vide++;
                    }
                } break;
        }
        if(vide==3) estvide=true;
        return estvide;
    }

    // fonction pour remlaire la matrice ....

    public void remplaire(int x, int y,int v,int posv){
        int i= x/ 40;
        int j=y/40;
        switch(posv) {
          case 0:  int L = j - 3;

            if (y < 350 && y > 125) {

                for (int k = L; k < j; k++) {
                    carte[k][i] = vect[v][k - L];
                }
            } break;
            case 1:     // modifier les valeur pour les adaptÃ© a la surface carte
                        int L1 = i - 1;
                        int j1=j-2;
                if (y < 400 && y > 70 && x>40 && x<260) {

                    for (int k = L1; k < i+2; k++) {
                        carte[j1][k] = vect[v][k - L1];


                    }
                } break;
            case 2:  int x2=j-3;
                int L2=0;


                if (y < 350 && y > 125) {
                   int k=j-1;
                    while(k>=x2){
                        carte[k][i] = vect[v][L2];
                        L2++;
                        k--;

                    }
                } break;
            case 3:     // modifier les valeur pour les adaptÃ© a la surface carte
                int x3 = i - 2;
                int L3 =0;
                int j3=j-2;
                if (y < 400 && y > 70 && x>40 && x<260) {

                    for (int k = i+1; k > x3; k--) {
                        carte[j3][k] = vect[v][L3]; L3++;


                    }
                } break;
              default:
        }
    }


    // dessin de la carte du jeu
    private void paintcarte(Canvas canvas) {
        for (int i = 0; i < carteHeight; i++) {
            for (int j = 0; j < carteWidth; j++) {
                switch (carte[i][j]) {
                    case CST_vide:
                        canvas.drawBitmap(vide, j * carteTileSize, i * carteTileSize, null);
                        break;
                    case CST_vert:
                        canvas.drawBitmap(vert_max, j * carteTileSize, i * carteTileSize, null);
                        break;
                    case CST_blue:
                        canvas.drawBitmap(blue_max, j * carteTileSize, i * carteTileSize, null);
                        break;
                    case CST_rouge:
                        canvas.drawBitmap(rouge_max, j * carteTileSize, i * carteTileSize, null);
                        break;
                    case CST_jaune:
                        canvas.drawBitmap(jaune_max, j * carteTileSize, i * carteTileSize, null);
                        break;
                    case CST_blanc:
                        canvas.drawBitmap(blanc_max, j * carteTileSize, i * carteTileSize, null);
                        break;
                    case CST_mauve:
                        canvas.drawBitmap(mauve_max, j * carteTileSize, i * carteTileSize, null);
                        break;

                }
            }
        }
    }


    private void paintvect(Canvas canvas) {
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (position[i] == 0) {

                    switch (vect[i][j]) {

                        case CST_vert:
                            canvas.drawBitmap(vert_min, depX[i] , depY[i]  + j * carteTileSizemin, null);
                            break;
                        case CST_blue:
                            canvas.drawBitmap(blue_min,  depX[i] , depY[i] + j * carteTileSizemin, null);
                            break;
                        case CST_rouge:
                            canvas.drawBitmap(rouge_min,  depX[i], depY[i] + j * carteTileSizemin, null);
                            break;
                        case CST_jaune:
                            canvas.drawBitmap(jaune_min,  depX[i], depY[i] + j * carteTileSizemin, null);
                            break;
                        case CST_blanc:
                            canvas.drawBitmap(blanc_min,  depX[i] , depY[i] + j * carteTileSizemin, null);
                            break;
                        case CST_mauve:
                            canvas.drawBitmap(mauve_min,  depX[i] , depY[i] + j * carteTileSizemin, null);
                            break;
                    }
                } else {
                    if (position[i] == 1) {

                        switch (vect[i][j]) {
                            case CST_vert:
                                canvas.drawBitmap(vert_min, depX[i]-carteTileSizemin+j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                break;
                            case CST_blue:
                                canvas.drawBitmap(blue_min,  depX[i]-carteTileSizemin+ j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                break;
                            case CST_rouge:
                                canvas.drawBitmap(rouge_min, depX[i]-carteTileSizemin+ j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                break;
                            case CST_jaune:
                                canvas.drawBitmap(jaune_min, depX[i]-carteTileSizemin+ j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                break;
                            case CST_blanc:
                                canvas.drawBitmap(blanc_min, depX[i]-carteTileSizemin + j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                break;
                            case CST_mauve:
                                canvas.drawBitmap(mauve_min, depX[i]-carteTileSizemin + j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                break;

                            //------------


                        }
                    } else {
                        if (position[i] == 2) {
                            switch (vect[i][j]) {
                                case CST_vert:
                                    canvas.drawBitmap(vert_min,  depX[i], depY[i]+75 - j * carteTileSizemin, null);
                                    break;
                                case CST_blue:
                                    canvas.drawBitmap(blue_min,  depX[i], depY[i]+75 - j * carteTileSizemin, null);
                                    break;
                                case CST_rouge:
                                    canvas.drawBitmap(rouge_min,  depX[i], depY[i]+75 - j * carteTileSizemin, null);
                                    break;
                                case CST_jaune:
                                    canvas.drawBitmap(jaune_min,  depX[i] , depY[i]+75 - j * carteTileSizemin, null);
                                    break;
                                case CST_blanc:
                                    canvas.drawBitmap(blanc_min,  depX[i] , depY[i]+75 - j * carteTileSizemin, null);
                                    break;
                                case CST_mauve:
                                    canvas.drawBitmap(mauve_min,  depX[i] , depY[i]+75 - j * carteTileSizemin, null);
                                    break;

                                //------------


                            }
                        } else {
                            switch (vect[i][j]) {
                                case CST_vert:
                                    canvas.drawBitmap(vert_min, depX[i] + carteTileSizemin - j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                    break;
                                case CST_blue:
                                    canvas.drawBitmap(blue_min, depX[i] + carteTileSizemin - j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                    break;
                                case CST_rouge:
                                    canvas.drawBitmap(rouge_min, depX[i] + carteTileSizemin - j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                    break;
                                case CST_jaune:
                                    canvas.drawBitmap(jaune_min, depX[i] + carteTileSizemin - j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                    break;
                                case CST_blanc:
                                    canvas.drawBitmap(blanc_min, depX[i] + carteTileSizemin - j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                    break;
                                case CST_mauve:
                                    canvas.drawBitmap(mauve_min, depX[i] + carteTileSizemin - j * carteTileSizemin, depY[i] + carteTileSizemin, null);
                                    break;

                                //------------
                            }

                        }
                    }
                }
            }
        }
    }
    // dessin du curseur du joueur


    // dessin des diamants


    // permet d'identifier si la partie est gagnee (tous les diamants ï¿½ leur place)
    private boolean isWon() {
        for (int i = 0; i < 4; i++) {
            //if (!IsCell(diamants[i][1], diamants[i][0], CST_zone)) {
            return false;
        }


        return true;


    }

    // dessin du jeu (fond uni, en fonction du jeu gagne ou pas dessin du plateau et du joueur des diamants et des fleches)
    private void nDraw(Canvas canvas) {
        canvas.drawRGB(48, 48, 48);
        if (isWon()) {

            //paintcarte(canvas);

        } else {
            paintcarte(canvas);
            paintvect(canvas);

        }

    }

    // callback sur le cycle de vie de la surfaceview
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        Log.i("-> FCT <-", "surfaceChanged " + width + " - " + height);
        initparameters();
    }

    public void surfaceCreated(SurfaceHolder arg0) {
        Log.i("-> FCT <-", "surfaceCreated");
    }


    public void surfaceDestroyed(SurfaceHolder arg0) {
        Log.i("-> FCT <-", "surfaceDestroyed");
    }

    /**
     * run (run du thread crï¿½ï¿½)
     * on endort le thread, on modifie le compteur d'animation, on prend la main pour dessiner et on dessine puis on libï¿½re le canvas
     */
    public void run() {
        Canvas c = null;
        while (in) {
            try {
                cv_thread.sleep(40);
                currentStepZone = (currentStepZone + 1) % maxStepZone;
                try {
                    c = holder.lockCanvas(null);
                    nDraw(c);
                } finally {
                    if (c != null) {
                        holder.unlockCanvasAndPost(c);
                    }
                }
            } catch (Exception e) {
                Log.e("-> RUN <-", "PB DANS RUN");
            }
        }
    }

    // verification que nous sommes dans le tableau
    private boolean IsOut(int x, int y) {
        if ((x < 0) || (x > carteWidth - 1)) {
            return true;
        }
        if ((y < 0) || (y > carteHeight - 1)) {
            return true;
        }
        return false;
    }

    //controle de la valeur d'une cellule
    private boolean IsCell(int x, int y, int mask) {
        if (carte[y][x] == mask) {
            return true;
        }
        return false;
    }

    // controle si nous avons un diamant dans la case
   /* private boolean IsDiamant(int x, int y) {
        for (int i=0; i< 4; i++) {
            if ((diamants[i][1] == x) && (diamants[i][0] == y)) {
                return true;
            }
        }
        return false;
    }*/

    // met ï¿½ jour la position d'un diamant
    /*private void UpdateDiamant(int x, int y, int new_x, int new_y) {
        for (int i=0; i< 4; i++) {
            if ((diamants[i][1] == x) && (diamants[i][0] == y)) {
                diamants[i][1] = new_x;
                diamants[i][0] = new_y;
            }
        }
    }*/
    // fonction permettant de recuperer les retours clavier



    // public boolean onKeyDown(int keyCode, KeyEvent event) {

    //	Log.i("-> FCT <-", "onKeyUp: "+ keyCode);
/*
        int xTmpPlayer	= xPlayer;
        int yTmpPlayer  = yPlayer;
        int xchange 	= 0;
        int ychange 	= 0;

        if (keyCode == KeyEvent.KEYCODE_0) {
        	initparameters();
        }

        if (keyCode == KeyEvent.KEYCODE_DPAD_UP) {
        	ychange = -1;
        }

        if (keyCode == KeyEvent.KEYCODE_DPAD_DOWN) {
        	ychange = 1;
        }

        if (keyCode == KeyEvent.KEYCODE_DPAD_LEFT) {
            xchange = -1;
        }

        if (keyCode == KeyEvent.KEYCODE_DPAD_RIGHT) {
            xchange = 1;
        }
	        xPlayer = xPlayer+ xchange;
	        yPlayer = yPlayer+ ychange;

	        if (IsOut(xPlayer, yPlayer) || IsCell(xPlayer, yPlayer, CST_block)) {
	            xPlayer = xTmpPlayer;
	            yPlayer = yTmpPlayer;
	        } else if (IsDiamant(xPlayer, yPlayer)) {
	            int xTmpDiamant = xPlayer;
	            int yTmpDiamant = yPlayer;
	            xTmpDiamant = xTmpDiamant+ xchange;
	            yTmpDiamant = yTmpDiamant+ ychange;
	            if (IsOut(xTmpDiamant, yTmpDiamant) || IsCell(xTmpDiamant, yTmpDiamant, CST_block) || IsDiamant(xTmpDiamant, yTmpDiamant)) {
	                xPlayer = xTmpPlayer;
	                yPlayer = yTmpPlayer;
	            } else {
	                UpdateDiamant(xTmpDiamant- xchange, yTmpDiamant- ychange, xTmpDiamant, yTmpDiamant);
	            }
	        }*/
    // return true;
    //}

    // fonction permettant de recuperer les evenements tactiles
    public boolean onTouchEvent(MotionEvent event) {
        final int y = (int) event.getY();
        final int x = (int) event.getX();
        // final int t = event.getAction();

        if (event.getAction() == MotionEvent.ACTION_DOWN) {
            Log.i("-> FCT <-", "onTouchEvent: ");

            if (y > 360 && x < 106) {
                // position[0] = Position(position[0]);
                depY[0] = y - 100;
                pv=position[0];
                place = 0;
                clic=true;

            } else if (event.getY() > 360 && event.getX() > 106 && event.getX() < 212) {
                //position[1] = Position(position[1]);
                depY[1] = y - 100;
                pv=position[1];
                place = 1;
                clic=true;
            } else if (event.getY() > 360 && event.getX() > 212) {
                // position[2] = Position(position[2]);
                depY[2] = y - 100;
                pv=position[2];
                place = 2;
                clic=true;
            }

        }
            if (event.getAction() == MotionEvent.ACTION_MOVE) {



                switch (place) {
                    case 0:
                        depY[0] = y - 130;
                        depX[0] = x;
                        break;
                    case 1:
                        depY[1] = y - 130;
                        depX[1] = x;

                        break;
                    case 2:
                        depY[2] = y - 130;
                        depX[2] = x;
                        break;
                }
            }


        if (event.getAction() == MotionEvent.ACTION_UP) {
            if (clic && Estvide(x, y, pv)) {
                remplaire(x, y, place, pv);
                suprimme();
                Changevect(place);

            }
                switch (place) {
                    case 0:
                        position[0] = Position(position[0]);
                        depY[0] = 330;
                        depX[0] = 35;
                        place = 3;
                        clic = false;
                        break;
                    case 1:
                        position[1] = Position(position[1]);
                        depY[1] = 330;
                        depX[1] = 141;
                        clic = false;
                        place = 3;

                        break;
                    case 2:
                        position[2] = Position(position[2]);
                        depY[2] = 330;
                        depX[2] = 247;
                        place = 3;
                        clic = false;
                        break;
                }


        }
        //return super.onTouchEvent(event);
        return true ;
        }

    }
